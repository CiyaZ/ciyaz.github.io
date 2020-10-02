# npm包管理器

npm是nodejs的包管理工具，不仅是nodejs，在浏览器客户端开发的前端项目也可以用npm进行管理，npm负责安装/管理第三方模块，解决依赖，发布第三方模块。本文总结了一些常用的npm命令。

## 查看npm版本和npm帮助 （npm help）

`npm -v`：查看npm版本

`npm help <command>`：查看npm帮助，例如：`npm help npm`，查看全局帮助，`npm help install`，查看npm install的帮助

## 安装（npm install）

`npm install npm -g`：更新到最新npm

`npm install`：在一个有package.json的目录里使用，会安装当前模块的所有依赖

### 安装新模块

**全局安装**：安装到node_modules的全局目录，将模块bin目录链接到系统可执行目录，用于全局使用的命令行工具安装，用法：`npm install <module> -g`

**本地安装**：安装到当前文件夹node_modules目录，用于一些当前工作目录的依赖库安装，用法：`npm install <module>`

注意：有些npm包，既是框架又有命令行工具，有些包则分开打包，其中十分复杂，建议安装一个npm包前，仔细阅读该包的使用手册，使用其推荐的安装方式。一般都是用本地安装。

可选参数（使用这两个参数前提是项目根目录有`package.json`）：

* `--save-dev`：安装的同时将这个依赖项写到`package.json`里的`devDependencies`里
* `--save`：安装的同时将这个依赖项写到`package.json`里的`dependencies`里

## 查看已安装的模块（npm ls）

`npm ls -g`：查看全局安装的模块

注意：该命令会同时分析依赖树，可能输出的列表非常长，可以用`npm ls -g > list.txt`

`npm ls`：查看当前目录可用的模块

注意：同上

## 查看模块注册信息（npm view）

`npm view <module>`

注意：该命令会输出缓存中最新的模块信息，而不是本地已经安装的，本地已安装的版本信息`npm list`即可查看

## 卸载模块（npm uninstall）

* 卸载本地安装的模块：`npm uninstall <module>`
* 卸载全局安装的模块：`npm uninstall <module> -g`

注：开发中，我们如果想要删除一个不用的依赖模块，更常见直观的做法是在`package.json`中删除该模块依赖，然后执行`npm install`。

## 更新模块（npm update）

`npm update <module>`

注意：

* 不指定-g选项，全局安装和本地安装的模块都会更新
* 该命令能自动修复缺失的依赖包
* 详细叙述参考`npm help update`

## 搜索模块（npm search）

`npm search [-l] <pattern>`

注意：

* 该命令会输出npm包缓存中所有包名和pattern匹配的模块
* -l选项：输出完整信息，不指定输出信息有很多省略号，不方便观看

## 缓存（npm cache）

`npm cache clean` 清理缓存，一般只用这个

## 运行 npm scripts（npm run）

我们项目中的`package.json`通常会定义一些`scripts`，调用各种工具完成打包、测试等工作，这些命令可以用`npm run` 调用，比如`npm run start`，表示调用`start`这个`scripts`配置。

注意：`npm run start`可以简写为`npm start`，这两个命令是一个意思。

## 发布模块

1. `npm init`：初始化package.json，根据CLI提示输入信息，即可自动创建package.json
2. `npm adduser`：创建npm账户，根据CLI提示，输入用户名，密码，邮箱
3. `npm publish`：发布当前目录下的模块

注：如果要发布到公共npm仓库，我们需要在`~/.npmrc`中把国内镜象配置暂时删掉

## 如何删除node_modules

`npm`使用体验非常差，最大的问题就是它会在工程中下载数以万计的小文件，不仅安装奇慢，而且删除都删不掉。

Windows下可以使用`Shift+Delete`键，这样能够跳过回收站，直接永久删除，Linux下直接`rm -r`即可。
