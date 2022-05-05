+++
author = "zs"
title = "webpack 原理与实践"
date = "2022-05-02"
lastmod = "2022-05-02"
description = ""
tags = [
    "webpack",
]
+++

## 如何利用插件机制横向扩展 webpack 的构建能力

插件机制是为了增强 webpack 在项目自动化构建方面的能力，通过上一讲我们知道 Loader 负责完成项目中各种资源模块的加载，从而实现整体项目的模块化，而 plugin 则是用来解决项目中除了资源模块打包以外的其他自动化工作。

首先介绍几个插件最常见的应用场景：
* 在打包之前自动清除 dist 目录（上次打包结果）；
* 自动生成应用所需要的 html 文件；
* 根据不同环境为代码注入类似 API 地址这种可能变化的部分；
* 拷贝不需要参与打包的资源文件到输出目录；
* 压缩 webpack 打包完成后输出的文件；
* 自动发布打包结果到服务器实现自动部署；

借助插件，我们可以轻松实现前端工程化中绝大多数经常用到的功能。

### 体验插件机制

我们先来体验几个最常见的插件，首先第一个就是用来自动清除输出目录的插件。

webpack 每次打包的结果都是直接覆盖到 dist 目录，而在打包之前，dist 目录就可能存在上一次打包的内容，如果我们再次打包，只能覆盖同名文件，已移除的资源文件就会累积在里面，最终部署上线就会有多余的文件，这显然是不合理的。

更为合理的做法就是在每次打包之前，自动清理 dist 目录，这样每次打包过后，dist 目录中就只有必要的文件。

**clean-webpack-plugin** 这个插件就实现了这一需求，它是一个第三方的 npm 包，首先要通过 npm 安装一下。

```
npm install clean-webpack-plugin --save-dev 
```

安装完后，回到 webpack 配置文件，然后导入 clean-webpack-plugin 插件，这个插件模块导出了一个叫做 CleanWebpackPlugin 的成员，我们把它结构出来。

```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
```

回到配置对象中，添加一个 plugin 属性，这个属性就是专门用来配置插件的地方，它是一个数组，添加一个插件就是在这个数组中添加一个元素。

绝大多数插件导出的都是一个类型，CleanWebpackPlugin 也不例外，使用它，就是通过这个类型创建一个实例，放入 plugins 数组中，

**webpack.config.js**
```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
  },
  plugins: [
    new CleanWebpackPlugin()
  ]
}
```

再次运行 webpack 进行打包，此时之前的打包结果就不会存在了，dist 目录中存放的就都是我们本次打包的结果。

一般来说，当我们有了某个自动化的需求后，可以先去找到一个合适的插件，然后安装，最后将它配置到 webpack 配置对象的 plugins 数组中。

#### 用于生成 HTML 的插件

除了自动清理 dist 目录，我们还有一个常见的需求，就是自动生成使用打包结果的 HTML，所谓使用打包结果指的是在 HTML 中自动注入 webpack 打包生成的 bundle。

在使用这个插件之前，我们的 HTML 文件一般都是通过硬编码的方式，单独存放在根目录下的，这种方式有两个问题：

项目发布时，我们需要同时发布根目录下的 HTML 文件和 dist 中的打包结果，而且上线后还要确保 HTML 代码中的资源文件路径是正确的。

如果打包结果输出的目录或者文件名称发生变化，那 HTML 代码中所对应的 script 标签也需要我们手动修改路径。

解决这两个问题最好的办法就是让 webpack 在打包的同时，自动生成对应的 HTML 文件，让 HTML 文件也参与到整个项目的构建过程。这样，在构建过程中，webpack 就可以自动将打包的 bundle 文件引入到页面中。

```js
npm install html-webpack-plugin --save-dev
```

安装完成后，回到配置文件，载入这个模块，不同于 clean-webpack-plugin，html-webpack-plugin 插件默认导出的就是插件类型，不需要再结构内部成员：

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
```

**webpack.config.js**
```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin()
  ]
}
```

再次运行打包命令，此时打包过程中就会自动生成一个 index.html 文件到 dist 目录。我们找到这个文件，可以看到文件中的内容就是一段使用了 bundle.js 的空白 HTML。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Bundle.js</title>
  </head>
  <body>
  <script type="text/javascript" src="bundler.js"></script></body>
</html>
```

到这里，webpack 就可以动态生成应用所需的 HTML 文件了，但是这里仍然存在一些需要改进的地方：
* 对于生成的 HTML 文件，页面 title 必须要修改；
* 需要自定义页面的一些 meta 标签和一些基础的 DOM 结构；

也就是说，还需要我们能充分自定义这个插件最终输出的 HTML 文件。

如果只是简单的自定义，我们可以通过修改 HtmlWebpackPlugin 的参数来实现。

我们回到 webpack 的配置文件中，这里我们给 HtmlWebpackPlugin 构造函数传入一个对象参数，用于指定配置选项。其中，title 属性设置的是 HTML 的标题，我们把它设置为 Webpack Plugin Simple。meta 属性需要以对象的形式设置页面中的元数据标签，这里我们尝试为页面添加一个 viewport 设置，

```js
// ./webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      meta: {
        viewport: 'width=device-width'
      }
    })
  ]
}
```

如果需要对 html 进行大量的自定义，更好的做法是在源代码中添加一个用于生成 html 的模版，然后让 html-webpack-plugin 插件根据这个模版去生成页面文件。

在 src 目录下新建一个 index.html 文件作为 html 文件的模版，然后根据我们的需要在这个文件中添加相应的元素。对于模版中动态的内容，可以使用 lodash 模版语法输出，模版中可以通过 **htmlWebpackPlugin.options** 访问这个插件的配置数据。

```html
<!-- ./src/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
  <div class="container">
    <h1>页面上的基础结构</h1>
    <div id="root"></div>
  </div>
</body>
</html>
```

在配置文件中，通过 template 属性指定所使用的模版，

```js
// ./webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      template: './src/index.html'
    })
  ]
}
```

再来添加一个 HtmlWebpackPlugin 实例用于创建一个 about.html 的页面文件，我们需要通过 filename 指定输出文件名，这个属性的默认值是 index.html，我们把它设置为 about.html，因此就可以输出多个 html 文件。

```js
// ./webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    // 用于生成 index.html
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      template: './src/index.html'
    }),
    // 用于生成 about.html
    new HtmlWebpackPlugin({
      filename: 'about.html'
    })
  ]
}
```

#### 用于复制文件的插件
在项目中还有一些不需要参与构建的静态文件，我们一般把这些文件放在 public 或者 static 目录中，我们希望打包时一并将这个目录下的所有文件复制到输出目录。

对于这种需求，可以使用 copy-webpack-plugin 插件来实现，这个插件的构造函数需要传入一个字符串数组，用于指定需要拷贝的文件路径。可以是一个通配符，也可以是一个目录或者文件的相对路径。

```js
new CopyWebpackPlugin({
  patterns: ['public'] // 需要拷贝的目录或者路径通配符
})
```

### 开发一个插件
webpack 的插件机制就是在开发中最常见的钩子机制。webpack 几乎在每一个环节都埋下了一个钩子，这样我们在开发插件的时候，通过往这些不同节点上挂载不同的任务，就可以轻松扩展 webpack 的能力。
