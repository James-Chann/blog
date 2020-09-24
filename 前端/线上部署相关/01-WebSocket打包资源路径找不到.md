# webpack run build 静态资源找不到404

问题现象：用vue框架开发的一个demo项目，在本地测试运行都是可以的，后来需要放到服务器上，用nginx做代理，结果客户说找不到打包后的静态资源，浏览器控ne制台错误代码404，请求控制台network报了一个css，js资源找不到404。于是开始找寻解决方法了。。。

因为服务器上线方式的调整，不会把项目放在根路径，https://192.168.10.35/static/css/app.f0b3e516f7ecc68501f83a83a9ae54a2.css 这个文件找不到，看看我们正常打包好的目录。

![](https://upload-images.jianshu.io/upload_images/13055508-621832f24baef3db.png?imageMogr2/auto-orient/strip|imageView2/2/w/196/format/webp)


正确的访问路径是：192.168.10.35/fcypt/static/css/app.149f36018149fcbe537f02cafdc6f047
config/index.js配置如图：

```javascript
build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist/index.html'),

    // Paths
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './',

    /**
     * Source Maps
     */

    productionSourceMap: true,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }
}
```

思来想去之前打包好的文件直接扔到nginx就可以使用，实在不清楚原因。后来经过运维小姐姐的排查后，总结了一下想想可能问题还是在前端打包时webpack配置路径发生了问题。凭着感觉尝试着改了下面一些代码（抱着试一试的感觉(●ˇ∀ˇ●)~）

```javascript
build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist/fcypt/index.html'),

    // Paths
    assetsRoot: path.resolve(__dirname, '../dist/fcypt'),
    assetsSubDirectory: 'static',

    /**
     * You can set by youself according to actual condition
     * You will need to set this if you plan to deploy your site under a sub path,
     * for example GitHub pages. If you plan to deploy your site to https://foo.github.io/bar/,
     * then assetsPublicPath should be set to "/bar/".
     * In most cases please use '/' !!!
     */
    // assetsPublicPath: '/vueAdmin-template/', // If you are deployed on the root path, please use '/'
    assetsPublicPath: '/fcypt/',

    /**
     * Source Maps
     */

    productionSourceMap: false,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }
```

这里需要注意assetsPublicPath:'/fcypt/' 末尾的斜杠一定要加，不然部分js打包后会出现https://192.168.10.35/fcyptstatic/css/app.149f36018149fcbe537f02cafdc6f047这样的情况。

看下打包好的目录，对比之后会发现多了一层fcypt目录，这个多出来的路径是index和assetsRoot这两个设置决定的。

然后放到服务器上，哈哈，圆满解决了~~

最后总结了一下~

问题的原因是服务器要配置多个项目，从而没有指定项目目录，因此需要在打包时对打包文件添加访问的项目名称，所以在配置打包路径是要加上项目名称。

webpack是真的不熟练呀，下面配上点vue cli默认webpack config／index.js的配置解释（怕自己忘了，好记性不如烂笔头嘛~）

```javascript
   var path = require('path') 
   module.exports = {
      build: { // production 环境
        env: require('./prod.env'), // 使用 config/prod.env.js 中定义的编译环境
        index: path.resolve(__dirname, '../dist/index.html'), // 编译输入的 index.html 文件
        assetsRoot: path.resolve(__dirname, '../dist'), // 编译输出的静态资源路径
        assetsSubDirectory: 'static', // 编译输出的二级目录
        assetsPublicPath: '/', // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
        productionSourceMap: true, // 是否开启 cssSourceMap
        // Gzip off by default as many popular static hosts such as
        // Surge or Netlify already gzip all static assets for you.
        // Before setting to `true`, make sure to:
        // npm install --save-dev compression-webpack-plugin
        productionGzip: false, // 是否开启 gzip
        productionGzipExtensions: ['js', 'css'] // 需要使用 gzip 压缩的文件扩展名
     },
     dev: { // dev 环境
       env: require('./dev.env'), // 使用 config/dev.env.js 中定义的编译环境
       port: 8080, // 运行测试页面的端口
       assetsSubDirectory: 'static', // 编译输出的二级目录
       assetsPublicPath: '/', // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
       proxyTable: {}, // 需要 proxyTable 代理的接口（可跨域）
       // CSS Sourcemaps off by default because relative paths are "buggy"
       // with this option, according to the CSS-Loader README
       // (https://github.com/webpack/css-loader#sourcemaps)
       // In our experience, they generally work as expected,
       // just be aware of this issue when enabling this option.
       cssSourceMap: false // 是否开启 cssSourceMap
    }
 }
```