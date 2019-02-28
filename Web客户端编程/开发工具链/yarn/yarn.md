# yarn包管理器

yarn（中文名：毛线）是一个node包管理工具，相比npm有更多好的特性，但是npm仍在发展，并不能说谁能取代谁。目前阶段，建议使用yarn替代npm作为项目的包管理工具。

注意：一个项目中，不要混用包管理工具。

## 安装

Windows下可以下载yarn的安装程序：[https://yarn.bootcss.com/docs/install/#windows-stable](https://yarn.bootcss.com/docs/install/#windows-stable)

Linux下可以通过APT等包管理工具安装yarn软件包。

## 使用

### 查看yarn帮助

我们没必要记住yarn的各种命令，大致有个印象，使用时查看帮助即可。

查询`yarn`命令帮助：

```
yarn -h
```

查询`yarn add`命令帮助：

```
yarn add -h
```

### 设置镜像

yarn和npm一样，也可以设置镜像，选择国内镜像，下载速度回比较快。

```
yarn config set registry https://registry.npm.taobao.org
```

### 项目依赖管理

我们的项目至少要有一个`package.json`，依赖管理才能正常工作，如果`package.json`已经写好了，直接运行`yarn`就可以自动下载依赖：

```
yarn
```

添加依赖，会自动写入`package.json`：
```
yarn add <依赖包名>
```

添加时，可选参数有`--dev`、`--peer`、`--optional`，这和npm是一样的。

### 使用npm的项目改为yarn

如果项目中使用过npm，那么是不可以直接混用yarn的。如果我们要使用yarn，最好删掉`node-modules`文件夹和`package-lock.json`，然后重新运行`yarn`下载依赖，yarn会生成`yarn.lock`。

### 执行npm scripts

和npm使用方法一致，比如`npm start`对应就是`yarn start`，`npm run build`对应就是`yarn run build`。yarn使用方式和npm基本是一样的，放心用就行了。

### 全局安装包

全局安装还是用npm，免得引起冲突。
