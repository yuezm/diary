# node docker 镜像构建

> 默认已经安装好 docker，且了解 node、npm、docker 基本使用。

想安装 docker 请点击 [docker 安装，官方文档](https://docs.docker.com/install/)

## 开始构建

### package.json

在 package.json 文件中，增加如下

```
...
"scripts": {
  "build:docker": "docker build -t $npm_package_name:$npm_package_version --build-arg APP_NAME=$npm_package_name .", // 构建docker镜像
  "startup:docker": "pm2-docker start startup.json --env production" // docker镜像内启动node
},
...
```

- 使用命令 `npm run build:docker` 构建 docker 镜像，使用 `npm run startup:docker`启动项目；
- 使用变量 `$npm_package_name:$npm_package_version` 读取 package.json 中的项目名及版本号，使之和 docker 镜像名及版本号对应；
- 向 Dockerfile 传入参数`APP_NAME=$npm_package_name`，在 Dockerfile 内需要使用；
- 构建完成后，存储到镜像仓库，阿里云有免费的镜像仓库 [阿里云容器镜像服务文档](https://help.aliyun.com/document_detail/60945.html?spm=5176.166170.863063.btn1cr2.3ba8217f4OiCAx)

### Dockerfile

编辑 Dockerfile

```
FROM node:10-alpine

ARG APP_NAME # 接收运行 build:docker 命令时的传参

ENV APP_NAME $APP_NAME \
    APP_ROOT /var/www/$APP_NAME \ # 确定项目运行目录
    LOG_PATH /var/log/$APP_NAME #确定项目日志目录

WORKDIR $APP_ROOT

COPY package*.json ./
RUN npm ci --only=production
COPY . .

EXPOSE 80

CMD ["npm","run","startup:docker"]
```

### .dockerignore

在 .dockerignore 文件中指定不需要 COPY 的文件、文件夹，类似于.gitignore

### 构建

上述完成后，在项目中，执行 `npm run build:docker` 则可以成功构建镜像

## node docker 基础镜像

### 基于现有的镜像

在 [hub.docker](https://hub.docker.com/_/node/)上可以找到很多的 node docker 镜像。
其中，xx:alpine 镜像是基于 Alpine Linux 构建的基础镜像，它非常小，小到连 shell 都没有，但是如果只是作为 node 运行环境，完全可以使用它。

### 自己编译

### 基于 alpine、jessie、stretch

在[github/docker-node](https://github.com/nodejs/docker-node)下找到对象版本的 Dockerfile,在 linux 环境下运行构建即可

### 基于 Ubuntu

在 ubuntu 基础镜像下，使用 apt-get 安装 node 即可。但是在 ubuntu:16.04 下 node 版本较低，想要安装较新版本 node，请参考 [github/nodesource](https://github.com/nodesource/distributions/blob/master/README.md)

### 基于 distroless

基于 google 提供的 distroless 构建，请参考 [github/GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless)

## 参考文档

[三个技巧，将 Docker 镜像体积减小 90%](https://www.infoq.cn/article/3-simple-tricks-for-smaller-docker-images)  
[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)  
[docker 官方文档](https://docs.docker.com/)  
[hub.docker](https://hub.docker.com/)
