新的征途
======

以下介绍，来源于 `webpack` [官网指南](http://webpack.github.io/docs/tutorials/getting-started/)的翻译/体验。    
段落后面的_斜体字_，是我加上的**个人理解**，请各位慧眼甄别靠谱程度。(●'◡'●)


- - - - - - - - - -


这是一个小指南，通过一个简单的例子来引导你。  
你将能学到：  

+ 如何安装 webpack
+ 如何使用 webpack
+ 如何使用 loaders
+ 如何使用 开发（调试）服务器


- - - - - - - - - -


开始使用 webpack
-------------

首先，在命令行输入 `npm install webpack -g` 安装 `webpack`  
> 这样就能在命令行中使用 webpack 了

接下来，在一个空文件夹中，新建这些文件：  
`entry.js`，输入内容  
```javascript
document.write("It works.");
```

还有 `index.html` 文件，输入内容  
```html
<html>
	<head>
		<meta charset="utf-8">
	</head>
	<body>
		<script type="text/javascript" src="bundle.js" charset="utf-8"></script>
	</body>
</html>
```

然后，在命令行中运行该命令： `webpack ./entry.js bundle.js`，这将会编译你的文件并创建一个 `bundle` 文件。  
如果顺利的话，你将能看到如下信息：  
```
Hash: ca188ee5789bb780fcec
Version: webpack 1.13.1
Time: 56ms
    Asset     Size  Chunks             Chunk Names
bundle.js  1.42 kB       0  [emitted]  main
   [0] ./entry.js 28 bytes {0} [built]
```

在你的浏览器中打开 `index.html`，将会看到 `It works.`  
*而一开始，是没有 `bundle.js` 文件的，这由刚才使用的 `webpack` 命令生成*

第二个文件
-------

下面，让我们在另一个文件中添加一些代码  
添加 `content.js` 文件，内容如下：  
```javascript
module.exports = "It works from content.js.";
```
并更新 `entry.js` 文件：
```javascript
// document.write("It works."); // 删除这行
document.write(require("./content.js"));
```

重新编译  
```
webpack ./entry.js bundle.js
```

再次刷新你的浏览器时，你会看到 `It works from content.js.`  
*代码中，多了这样一段： function(module, exports, \_\_webpack\_require\_\_) {...}*  

> webpack 会分析你的 entry 文件所依赖的其它文件。同时这些文件（被成为 模块）也会被包含在 `bundle.js` 中。
> webpack 会给每个模块 一个唯一的id，并且在 `bundle.js` 文件中（以可理解的方式？？）保存所有模块的id。
> 但在启动时，只有 entry 模块会被执行。另外，（？？指bundle.js的作用？）还有 一个小的运行环境 提供了 `require` 函数 并执行其依赖。

- - - - - - - - - -

第一个 加载器(loader)
----------------

我们想要在应用程序中添加一个 css文件。

但原生的webpack（指没有安装插件时）只能处理 javascript 文件，所以我们需要 `css-loader` 来处理 CSS 文件。
并且我们也需要 `style-loader` 来应用 CSS 文件中的样式。 

运行 `npm install css-loader style-loader` 来安装这些加载器（这些加载器需要以本地模式安装，不必加 `-g` 全局安装标识）。
这将会在你的文件夹中创建 `node_modules` 目录 以存放 这些加载器。

我们开始学习吧：  
添加 `style.css` 文件，内容如下：  
```css
body {
	background: yellow;
}
```

更新 `entry.js` 文件  
```javascript
require("!style!css!./style.css"); // 这行是新增的
document.write(require("./content.js"));
```

重新编译并刷新你的浏览器，你会看到应用程序的背景变为黄色了。  
*bundle.js里面多了很多代码呢，并且将css文件的内容，以<style\>标签的形式，动态插入到html页面中了（index.html文件并没有变化）*

> By prefixing loaders to a module request, the module went through a loader pipeline.（这句我不会翻译 (-__-)b）
这些加载器以特殊的方式转换内容。即以javascript模块为结果 来应用这些转换。

- - - - - - - - - -

绑定加载器
-------
我们并不想写这么长的 requires ： `require("!style!css!./style.css");` 。  
那么我们可以为加载器 绑定文件后缀，这样我们就只需写： `require("./style.css")`

更新 `entry.js` 文件  
```javascript
// require("!style!css!./style.css"); // 删除原来这行
require("./style.css"); // 这行是新增的
document.write(require("./content.js"));
```

接着使用下面的命令编译：  
```
webpack ./entry.js bundle.js --module-bind 'css=style!css'
```
> 有些（运行）环境可能需要双引号，如 `–module-bind “css=style!css”`

你还是可以看到相同结果的。

*重点注意，如果你在命令行中遇到如下红色的错误信息（或其他错误信息），请试试将编译命令的单引号换成双引号*  
```
ERROR in ./style.css
Module parse failed: C:\...\xxx\style.css Unexpected token (1:5)
You may need an appropriate loader to handle this file type.
```
*我在第二次测试的时候就遇到了！试着换双引号，果然好了 ╮(╯-╰)╭*

- - - - - - - - - -

配置文件
------
让我们来把（命令行）配置参数 移至 配置文件中：  
添加 `webpack.config.js` 文件，像下面这样  
```javascript
module.exports = {
	entry: "./entry.js",
	output: {
		path: __dirname,
		filename: "bundle.js"
	},
	module: {
		loaders: [
			{ test: /\.css$/, loader: "style!css" }
		]
	}
};
```

现在我们可以运行这个命令：  

```
webpack
```

来编译：  
```
Hash: 920dfbaa4880fc425a89
Version: webpack 1.13.1
Time: 1640ms
    Asset     Size  Chunks             Chunk Names
bundle.js  11.8 kB       0  [emitted]  main
   [0] ./entry.js 109 bytes {0} [built]
   [5] ./content.js 45 bytes {0} [built]
    + 4 hidden modules
```

> webpack 命令行 将会尝试在 当前目录中加载 `webpack.config.js` 文件。

- - - - - - - - - -

美化输出
------
随着项目的增长，编译时间可能会有点久。因此我们希望进度条能展示更多种类，比如颜色。。。  
我们可以这样实现：  
```
webpack --progress --colors
```

- - - - - - - - - -

监测模式
------
我们不想要 每一个改变都需要重新手动编译。。。  
```
webpack --progress --colors --watch
```
Webpack 能够缓存未改变的模块 并输出被编译的文件。  
> 当使用监测模式时，webpack 为所有文件都添加了“文件观察者”（编译线程中会用到的）。
（这个“文件观察者”）如果监测到（文件）有任意改变，则重新执行编译。
当启用缓存时，webpack 会在内存中保存所有模块，而当（文件内容）没有改变时，会直接使用这些缓存模块。

*感觉最后一句翻译得怪怪的 (￣▽￣)"*

- - - - - - - - - -

开发（调试）服务器
-------------

使用开发服务器无疑更好。  
```
npm install webpack-dev-server -g
```

```
webpack-dev-server --progress --colors
```

这会在 localhost:8080 启动一个 提供静态文件（自动编译后的）的小型便捷服务器。
且当 bundle 重新编译之后 会自动刷新浏览器页面（使用了SockJS吧）。
在你的浏览器中打开 [http://localhost:8080/webpack-dev-server/bundle](http://localhost:8080/webpack-dev-server/bundle) 看看吧。

> 开发服务器使用了 webpack 的监测模式。
它会将服务器及（编译后的）结果文件保持在内存中，而不是将结果文件写入到硬盘中。
