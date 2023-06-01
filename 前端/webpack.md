# 一. 简介

## 1. 概述

webpack 用于编译 JavaScript 模块。一旦完成[安装](https://www.webpackjs.com/guides/installation)，你就可以通过 webpack 的 [CLI](https://www.webpackjs.com/api/cli) 或 [API](https://www.webpackjs.com/api/node) 与其配合交互。如果你还不熟悉 webpack，请阅读[核心概念](https://www.webpackjs.com/concepts)和[打包器对比](https://www.webpackjs.com/comparison)，了解为什么你要使用 webpack，而不是社区中的其他工具。

本质上，webpack 是一个现代 JavaScript 应用程序的**静态模块打包器(module bundler)**。当 webpack 处理应用程序时，它会递归地构建一个**依赖关系图(dependency graph)**，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 **bundle**。

### 1.1. 基本概念

从 webpack v4.0.0 开始，可以直接使用默认配置, 而不用引入一个配置文件。然而，webpack 仍然还是[高度可配置的](https://www.webpackjs.com/configuration)。在开始前你需要先理解四个**核心概念**：

#### 1) 入口 entry

**入口起点(entry point)**指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

可以通过在 [webpack 配置](https://www.webpackjs.com/configuration)中配置 `entry` 属性，来指定一个入口起点（或多个入口起点）。默认值为 `./src`。

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

#### 2) 输出 output

`output` 属性告诉 webpack 在哪里输出它所创建的 **bundles**，以及如何命名这些文件，默认值为 `./dist`。

基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个 `output` 字段，来配置这些处理过程：

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

#### 3) 加载器 loader

`loader` 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效[模块](https://www.webpackjs.com/concepts/modules)，然后你就可以利用 webpack 的打包能力，对它们进行处理。

本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

在 webpack 的配置中 **loader** 有两个目标：

1. `test` 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
2. `use` 属性，表示进行转换时，应该使用哪个 loader。

```js
const path = require('path');

const config = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

module.exports = config;
```

webpack 将**动态打包(dynamically bundle)**所有依赖项（创建所谓的[依赖图(dependency graph)](https://www.webpackjs.com/concepts/dependency-graph)）。**每个模块都可以明确表述它自身的依赖**，webpack 将避免打包未使用的模块。

webpack 最出色的功能之一就是, 可以通过 `loader` 引入任何其他类型的文件。也就是说，以上列出的那些 JavaScript 的优点（例如显式依赖），同样可以用来构建网站或 web 应用程序中的所有非 JavaScript 内容。

#### 4) 插件 plugins

`loader` 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。[插件接口](https://www.webpackjs.com/api/plugins)功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建它的一个实例。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

const config = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

#### 5) 模式

通过选择 `development` 或 `production` 之中的一个，来设置 `mode` 参数，你可以启用相应模式下的 webpack 内置的优化

```js
module.exports = {
  mode: 'production'
};
```

## 2. 安装

### 2.1. 局部安装

```shell
npm install --save-dev webpack
npm install --save-dev webpack@<version>
npm install --save-dev webpack-cli
```

### 2.2. 全局安装

>  不推荐全局安装 webpack。这会将你项目中的 webpack 锁定到指定版本，并且在使用不同的 webpack 版本的项目中，可能会导致构建失败。

# 二. 项目开发

## 1. 目录结构

## 2. 执行打包

```shell
npx webpack
```

执行 `npx webpack`，会将我们的脚本作为[入口起点](https://www.webpackjs.com/concepts/entry-points)，然后 [输出](https://www.webpackjs.com/concepts/output) 为 `main.js`。

Node 8.2+ 版本提供的 `npx` 命令，可以运行在初始安装的 webpack 包(package)的 webpack 二进制文件（`./node_modules/.bin/webpack`）

### 2.1. 模块

`ES2015` 中的 `import` 和 `export` 语句已经被标准化。虽然大多数浏览器还无法支持它们，但是 webpack 却能够提供开箱即用般的支持。

事实上，webpack 在将代码“转译”，以便旧版本浏览器可以执行。除了 `import` 和 `export`，webpack 还能够很好地支持多种其他模块语法，更多信息请查看[模块 API](https://www.webpackjs.com/api/module-methods)。

注意，webpack 不会更改代码中除 `import` 和 `export` 语句以外的部分。如果你在使用其它 ES2015 特性，请确保在 webpack 的 loader 系统中使用了合适的转译器, 如`Babel`。

## 3. 配置文件

在 webpack 4 中，可以无须任何配置使用，然而大多数项目会需要很复杂的设置，这就是为什么 webpack 仍然要支持 [配置文件](https://www.webpackjs.com/concepts/configuration)。

webpack 的配置文件，是导出一个对象的 **JavaScript 文件**, 是标准的 **Node.js CommonJS 模块**。

### 3.1. 指定配置文件

- 如果 `webpack.config.js` 存在，则 `webpack` 命令将默认选择使用它。

- 如果使用其他名称的配置文件, 需要在命令中指定配置文件的名称

  ```shell
  npx webpack --config webpack.config.dev.js
  ```

### 3.2. npm 脚本

使用命令行来运行 webpack 并不是很方便, 尤其是需要使用特定参数时, 因此, 可以使用 npm 脚本来创建快捷方式

只需要为在 `package.json` 文件中的 `scripts` 属性中添加名称和具体的命令

```json
{
  // ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "自定义名称": "具体的命令"
  },
	// ...
}
```

然后我们就可以通过 npm 快捷方式来执行我们的具体命令

```shell
npm run 自定义的名称
```

执行npm脚本时, 可以通过 `--` 双横杠, 将其他命令行参数传递到预设脚本中

```shell
npm run build -- --colors
# 实际上会执行
npx webpack --colors
```







