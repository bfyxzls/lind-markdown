# 一. NPM 简介

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。 
- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。 
- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

 由于新版的nodejs已经集成了npm，所以之前npm也一并安装好了

查看当前版本

```shell
npm -v
```

## 全局配置

新建两个文件夹

- `D:\nodejs\node_cache`
- `D:\nodejs\node_global`

```cmd
npm config set prefix "D:\Java\nodejs\node_global"
npm config set cache "D:\Java\nodejs\node_cache"
```

在系统变量中新增

 `NODE_PATH =  D:\nodejs\node_global\node_modules`

在用户变量中, 将

`Path  =  C:\Users\用户名\AppData\Roaming\npm `

修改为

 `Path =  D:\nodejs\node_global` 

测试全局安装, 确认安装到自定义的路径中

`npm install express -g`

配置淘宝的 npm 镜像

# 二. 项目开发

## 1. 项目目录结构

## 2. package.json

## 3. package-lock.json



# 三. npm 模块管理

## 安装模块

 npm 的包安装分为本地安装（local）、全局安装（global）两种，  差别只是有没有 `-g` 而已

```shell
# 本地安装
npm install <module_name>
# 全局安装
npm install <module_name> -g
```

本地安装时, 默认将包放在 `<当前目录>/node_modules` 下, 如果不存在则自动创建目录. 本地安装的包, 可以在项目中使用 `require()` 来引入并使用.

全局安装时, 默认将包放在node 的安装目录, 上面已修改为 `D:\nodejs\node_global\node_modules` 中. 全局安装的包, 可以直接在命令行里使用.

如果希望具备两者的功能, 需要在两个地方安装, 或者使用 `npm link`

使用淘宝 npm 镜像

由于官方服务器访问速度较慢, 国内一般使用淘宝镜像

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装 cnpm 后, 就可以使用淘宝镜像安装模块, 用法与 npm 完全一样

```shell
# 本地安装
cnpm install <module_name>
# 全局安装
cnpm install <module_name> -g
```

除了使用 cnpm 替代 npm, 也可以直接将 npm 的目标路径设为淘宝镜像服务器的路径, 这样使用直接使用 npm 命令也会从淘宝镜像服务器获取资源

- 使用命令修改

  ```shell
  npm config set registry https://registry.npm.taobao.org
  ```

- 或直接修改配置文件 

  打开 `[USER_HOME]/.npmrc` 文件, 增加以下配置

  ```properties
  registry=https://registry.npm.taobao.org
  ```

### 安装参数

```shell
# 安装模块到项目目录下
npm install moduleName

# -g 的意思是将模块安装到全局，具体安装到磁盘哪个位置，要看 npm config prefix 的位置。
npm install -g moduleName

# -save 的意思是将模块安装到项目目录下，并在package文件的dependencies节点写入依赖。
npm install -save moduleName

# -save-dev 的意思是将模块安装到项目目录下，并在package文件的devDependencies节点写入依赖。
npm install -save-dev moduleName
```





## 管理模块

```shell
# 查看全局安装的模块
$ npm list -g

# 查看模块版本号
$ npm list grunt

# 卸载模块
$ npm uninstall express

# 更新模块
$ npm update express

# 搜索模块
$ npm search express
```

## 创建模块

创建模块需要使用 `package.json`, 用来描述模块信息

```shell
$ npm init
$ npm adduser
$ npm publish
```

