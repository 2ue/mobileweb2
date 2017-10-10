# 个人尝试搭建一个快速开发移动站的架子2

> 基于**[mobileweb](https://github.com/whidy/mobileweb)**进行改动，[mobileweb2](https://github.com/whidy/mobileweb2)未使用PostCSS进行处理，而是采用了比较简单的模式，webpack3 + sass + [lib-flexible 0.3.2](https://github.com/amfe/lib-flexible/tree/master) + [px2rem](https://github.com/songsiqi/px2rem)两者均采用rem单位来进行适配处理，均解决retina屏下了1px边框问题（我仅测试了iPhone部分机型的DPR2.0下情况和少量安卓机型）。

### 搭建环境及相关配置

- **webpack 3.x**，需要loader及说明
- css-loader, style-loader 加载css文件
- expose-loader 暴露全局例如jquery
- url-loader 样式文件内的图片等资源
- file-loader 字体等资源

### 使用库和主要插件

- jquery
- normalize
- [lib-flexible 0.3.2](https://github.com/amfe/lib-flexible/tree/master)
- [px2rem](https://github.com/songsiqi/px2rem)

### 要解决一些问题

自适应这里采用了旧版的flexible，并通过px2rem来进行单位转换，关于样式中的px值是否转换为rem或者输出多种对应不同dpr的px值，请查看插件说明进行对应的注释，例如``/*no*/``和``/*px*/``。这里有一点需要说明的是，与**[mobileweb](https://github.com/whidy/mobileweb)**不同的是，旧版的flexible具有最大宽度1080(540*dpr)的问题？也就是说当屏幕宽度大于1080的时候，两边会留出空白，而无法占满屏幕？截取一段flexible代码：

```javascript
function refreshRem() {
  var width = docEl.getBoundingClientRect().width;
  if (width / dpr > 540) {
    width = 540 * dpr;
  }
  var rem = width / 10;
  docEl.style.fontSize = rem + 'px';
  flexible.rem = win.rem = rem;
}
```

这里有个小小的建议就是给body加上一段居中样式：

```css
body {
  max-width: 750px; /* 设计稿最大宽度 */
  margin: 0 auto;
}
```

这样当设备宽度大于设计稿的宽度时，则整体页面居中，更加美观。（再次强调mobileweb中用的最新的flexible会自动扩展到满屏，不存在该问题。）

### 附加：关于webpack配置写法参考

```javascript
module: {
    rules: [{
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.scss$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: [{
            loader: "css-loader"
          }, {
            loader: "px2rem-loader",
            options: {
              remUnit: 75,
              threeVersion: true
            }
          }, {
            loader: "sass-loader"
          }, ]
        })
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [{
          loader: 'url-loader',
          options: {
            limit: 4096
          }
        }]
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          'file-loader'
        ]
      },
      {
        test: require.resolve('jquery'),
        use: [{
          loader: 'expose-loader',
          options: 'jQuery'
        }, {
          loader: 'expose-loader',
          options: '$'
        }]
      }
    ]
  },
```

主要是px2rem-loader这里的对px2rem的相关配置，我这里设计稿750，因此设定75，其他参数可自行参考文档。

###运行方法

请依次分布执行

```javascript
npm install
npm run start
npm run build:dev
```

打开浏览器访问http://localhost:9000/src/

临时备注:我也看过大漠的[再聊移动端页面的适配](http://www.w3cplus.com/css/vw-for-layout.html)这篇文章,不过是否值得广泛使用还在研究中,所以等我目前还是比较倾向于旧的成熟一些的方案,这个有空我会进一步研究并记录成果~
