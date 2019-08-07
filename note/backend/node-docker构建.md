# node docker 镜像构建

## 安装 docker

- 苹果系统，直接下载安装 Docker.dmg，下载地址：[Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)。
- 其他系统可以参照官方文档安装：[docker 安装，官方文档](https://docs.docker.com/install/)

## 新建项目

### 创建项目

```
mkdir docker-demo
npm init
npm i koa -S
touch index.js
```

### 创建简单服务器

```
// ------ index.js ------
const Koa = require('koa');
const app = new Koa();

app.use(ctx => {
  ctx.body = 'Hello docker';
});
app.listen(8080);
```

## 开始构建 docker

### 编写 package.json

在 package.json 文件中，增加如下

```
...
"scripts": {
  "build": "docker build -t $npm_package_name:$npm_package_version --build-arg APP_NAME=$npm_package_name .", // 构建docker镜像
  "startup": "node index.js" // 容器内启动node，使用pm2时需要注意，不能后台启动，可以使用pm2-docker命令
},
...
```

- 使用命令 `npm run build` 构建 docker 镜像，使用 `npm run startup`启动项目；
- 使用变量 `$npm_package_name:$npm_package_version` 读取 package.json 中的项目名及版本号，使之和 docker 镜像名及版本号对应；
- 向 Dockerfile 传入参数`APP_NAME=$npm_package_name`，在 Dockerfile 内需要使用；
- 构建完成后，存储到镜像仓库，阿里云有免费的镜像仓库 [阿里云容器镜像服务文档](https://help.aliyun.com/document_detail/60945.html?spm=5176.166170.863063.btn1cr2.3ba8217f4OiCAx)

### 编写 Dockerfile

```
FROM node:10

ARG APP_NAME

ENV APP_NAME $APP_NAME
ENV APP_ROOT /var/www/$APP_NAME
ENV LOG_PAT H /var/log/$APP_NAME

WORKDIR $APP_ROOT

COPY package*.json ./
RUN npm ci --only=production
COPY index.js ./

EXPOSE 8080

CMD ["npm","run","startup"]
```

### 编写 .dockerignore

在 .dockerignore 文件中指定不需要 COPY 的文件、文件夹，类似于.gitignore

### 构建

上述完成后，在项目中，执行 `npm run build:docker` 则可以成功构建镜像，使用 `docker image ls` 查看是否构建完成  
![](https://public.keven.work/docker-size.png)

## 运行

在命令行中输入如下命令，运行该镜像 `docker run -p 8080:8080 docker-demo:1.0.0`,在浏览器中输入 `localhost:8080` 来查看是否运行，至此简单 docker 镜像构建完毕。

## 杂谈

### 减小镜像体积

使用 `docker image ls` 查看构建好的镜像，如图所示  
![](https://public.keven.work/docker-size.png)  
会发现镜像体积比较大，下面介绍如何减小镜像体积

#### 减少项目文件

- 使用 .dockerignore 减少复制的文件
- 使用 npm install 时，注意使用生产环境，减少 npm 包的体积

#### 压缩构建阶段

- Dockerfile 中每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建,每增加一层也会增加镜像大小，所以将指令合并，也可以减少体积。
- 应当注意，该层添加的文件并不会在下一层被删除，则需要保证每一层只添加需要的文件，其他应该被清理掉

```dockerfile
ENV APP_NAME=$APP_NAME \
    APP_ROOT=/var/www/$APP_NAME \
    LOG_PATH=/var/log/$APP_NAME
```

#### 使用体积较小的基础镜像

```dockerfile
FROM node:10-alpine
```

这是优化后的大小  
![](https://public.keven.work/docker-size-small.png)

### node 基础镜像

在 Dockerfile 中我们命令是 `FROM node:10`和 `FROM node:10-alpine`。使用了 node 基础镜像，你也可以使用其他的 docker 镜像，在 [hub.docker](https://hub.docker.com/_/node/)上可以找到很多的 node 镜像。当然你可以自己编译基础镜像，如下所示

#### 基于 alpine、jessie、stretch 编译

在[github/docker-node](https://github.com/nodejs/docker-node)下找到对象版本的 Dockerfile,在 linux 环境下运行构建即可，这里强调下 **linux 环境**，我尝试在 mac 中编译，会报错，但是在 ubuntu 中正常编译。

#### 基于 Ubuntu

在 ubuntu 基础镜像下，使用 apt-get 安装 node 即可。但是在 ubuntu:16.04 下 apt-get 的 node 版本较低，想要安装较新版本 node，请参考 [github/nodesource](https://github.com/nodesource/distributions/blob/master/README.md)

#### 基于 distroless

基于 google 提供的 distroless 构建，请参考 [github/GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless)

参考文档

[三个技巧，将 Docker 镜像体积减小 90%](https://www.infoq.cn/article/3-simple-tricks-for-smaller-docker-images)  
[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)  
[docker 官方文档](https://docs.docker.com/)  
[hub.docker](https://hub.docker.com/)
