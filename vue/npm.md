# npm

## 安装

npm 在按照 Node.js 时会连带被安装。但有可能不是最新版本，需要 `npm install npm@latest -g` 升级到最新版本。

## 基本命令

```
# 查看 npm 命令列表
$ npm help

# 查看各个命令的简单用法
$ npm -l

# 查看 npm 的版本
$ npm -v

# 查看 npm 的配置
$ npm config list -l
```

## npm init初始化package.json文件

用来初始化生成`package.json`文件。在这个过程中会向用户提问一系列问题，如果你觉得不用修改默认配置，一路回车就可以了。

如果使用了`-f`（代表`force`）、`-y`（代表`yes`），则跳过提问阶段，直接生成一个新的`package.json`文件。

## npm set 设置环境变量

```
  $ npm set init-author-name 'Your name'
  $ npm set init-author-email 'Your email'
  $ npm set init-author-url 'http://yourdomain.com'
  $ npm set init-license 'MIT'
```

上面命令等于为`npm init`设置了默认值，以后执行`npm init`的时候，`package.json`的作者姓名、邮件、主页、许可证字段就会自动写入预设的值。这些信息会存放在用户主目录的`~/.npmrc`文件，使得用户不用每个项目都输入。如果某个项目有不同的设置，可以针对该项目运行`npm config`。

```bash
  $ npm set save-exact true
```

上面命令设置加入模块时，`package.json`将记录模块的确切版本，而不是一个可选的版本范围。

## npm list

`npm list`命令以树型结构列出当前项目安装的所有模块，以及它们依赖的模块。

```php
npm list
npm list -global
npm list vue
```

加上global参数，会列出全局安装的模块。

## npm install

`Node`模块采用`npm install`命令安装。

每个模块可以“全局安装”，也可以“本地安装”。“全局安装”指的是将一个模块安装到系统目录中，各个项目都可以调用。一般来说，全局安装只适用于工具模块，比如`eslint`和`gulp`。“本地安装”指的是将一个模块下载到当前项目的`node_modules`子目录，然后只有在项目目录之中，才能调用这个模块

```csharp
# 本地安装
$ npm install <package name>

# 全局安装
$ sudo npm install -global <package name>
$ sudo npm install -g <package name>

# 也支持直接输入Github代码库地址
$ npm install git://github.com/package/path.git
$ npm install git://github.com/package/path.git#0.1.0

# 强制重新安装
$ npm install <packageName> --force

# 如果你希望，所有模块都要强制重新安装，那就删除node_modules目录，重新执行npm install
$ rm -rf node_modules
$ npm install
```

## 安装不同版本

1. install 命令总是安装模块的最新版本，如果要安装模块的特定版本，可以在模块名后面加上@和版本号。
2. --save在安装的同时将修改保存入package.json
3. --save-dev只在开发时才需要（开发依赖）

```ruby
$ npm install sax@latest
$ npm install sax@0.1.1
$ npm install sax@">=0.1.0 <0.2.0"

# 如果使用--save-exact参数，会在package.json文件指定安装模块的确切版本
$ npm install readable-stream --save --save-exact

$ npm install sax --save
$ npm install node-tap --save-dev
# 或者
$ npm install sax -S
$ npm install node-tap -D

# 如果要安装beta版本的模块，需要使用下面的命令
# 安装最新的beta版
$ npm install <module-name>@beta (latest beta)
# 安装指定的beta版
$ npm install <module-name>@1.3.1-beta.3

# npm install默认会安装dependencies字段和devDependencies字段中的所有模块，如果使用--production参数，可以只安装dependencies字段的模块
$ npm install --production
# 或者
$ NODE_ENV=production npm install
```

## 避免系统权限

默认情况下，`Npm`全局模块都安装在系统目录（比如`/usr/local/lib/`），普通用户没有写入权限，需要用到`sudo`命令。这不是很方便，我们可以在没有`root`权限的情况下，安装全局模块。

首先，在主目录下新建配置文件`.npmrc`，然后在该文件中将`prefix`变量定义到主目录下面。

```jsx
  prefix = /home/yourUsername/npm
```

然后在主目录下新建`npm`子目录

```undefined
$ mkdir ~/npm
```

此后，全局安装的模块都会安装在这个子目录中，`npm`也会到`~/npm/bin`目录去寻找命令。

最后，将这个路径在`.bash_profile`文件（或`.bashrc`文件）中加入`PATH`变量。

```bash
export PATH=~/npm/bin:$PATH
```

## npm update

npm update命令可以更新本地安装的模块

```ruby
# 升级当前项目的指定模块
$ npm update [package name]

# 升级全局安装的模块
$ npm update -global [package name]
```

它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。

使用`-S`或`--save`参数，可以在安装的时候更新`package.json`里面模块的版本号。

注意，从`npm v2.6.1`开始，npm update只更新顶层模块，而不更新依赖的依赖，以前版本是递归更新的。如果想取到老版本的效果，要使用下面的命令。

```ruby
$ npm --depth 9999 update
```

## npm uninstall

npm uninstall命令，卸载已安装的模块

```ruby
$ npm uninstall [package name]

# 卸载全局模块
$ npm uninstall [package name] -global
```

## npm run

npm 不仅可以用于模块管理，还可以用于执行脚本。package.json 文件有一个 scripts 字段，可以用于指定脚本命令，供npm直接调用。

npm run 命令会自动在环境变量 $PATH 添加 node_modules/.bin 目录，所以 scripts 字段里面调用命令时不用加上路径，这就避免了全局安装 NPM 模块。

npm run 如果不加任何参数，直接运行，会列出 package.json 里面所有可以执行的脚本命令。

npm内置了两个命令简写，npm test 等同于执行 npm run test，npm start 等同于执行 npm run start。

```ruby
$ npm i eslint --save-dev
```