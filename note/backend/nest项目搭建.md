# Nest 初体验

## 初始化项目

### 使用 Nest CLI 构建项目

```
$ npm i -g @nestjs/cli
$ nest new nest-example
```

### 项目目录拆分

![](https://public.keven.work/nest-dir.png)

## 基础配置

### 环境变量

引入 **dotenv** 包，读取项目根目录 _.env_ 文件

- _.env_ 文件不提交到 git 仓库，仓库维护 _.env-example_ 文件，在开发环境下由开发人员重命名为 _.env_ 文件，读取开发配置
- 测试、正式环境的 _.env_ 文件由配置仓库统一维护，在启动工程前自动向配置仓库拉取，放置于根目录下

[github.com/dotenv](https://github.com/motdotla/dotenv)

### 日志

引入 **winston** 包，由 _src/provider/service/log.service.ts_ 维护，在 _src/app.module.ts_ 添加为全局注册

- 日志路径由 `LOG_PATH` 环境变量维护，由 _.env_ 或 _Dockerfile_ 文件指定
- 日志切割：日志按天切割，日志格式为 \`YYYY-MM-DD HH:mm:ss [${level}] \${error_message}\`

[nestjs 文档 / 日志](https://docs.nestjs.cn/5.0/techniques?id=%e6%97%a5%e5%bf%97)  
[github.com/winston](https://github.com/winstonjs/winston)  
[github.com/winston-daily-rotate-file](https://github.com/winstonjs/winston-daily-rotate-file)

### 登录校验

由 _src/provider/guards/auth.guards.ts_ 维护，在 _src/app.module.ts_ 添加为全局注册

- 如果未登录或登录失效，则返回 `HttpStatus.UNAUTHORIZED`
- 如果是 gRPC 服务，则需要抛出 `RpcException`，而非抛出 `HttpException`

[nestjs 文档 / 授权看守卫](https://docs.nestjs.cn/5.0/guards?id=%e6%8e%88%e6%9d%83%e7%9c%8b%e5%ae%88%e5%8d%ab)

### 请求参数校验

由 nest 的 `ValidationPipe` 维护，在 _src/app.module.ts_ 添加为全局注册。具体参数校验由 _src/core/dto/\*.dto.ts_ 各自维护，Demo 如下

```
// ------- src/core/dto/test.dto.ts -------
import { IsInt } from "class-validator";
import { Type } from "class-transformer";

export class TestDto {
  @IsInt()             // 必填校验，id必须是int
  @Type(() => Number) // 转换id类型，由string转换为number
  id: number;
}
```

[nestjs 文档 / 类验证器](https://docs.nestjs.cn/5.0/pipes?id=%e7%b1%bb%e9%aa%8c%e8%af%81%e5%99%a8%ef%bc%88class-validator%ef%bc%89)  
[github/class-validator](https://github.com/typestack/class-validator)  
[github/class-transformer](https://github.com/typestack/class-transformer)

### 响应值处理

由 _src/provider/interceptor/response.interceptor.ts_ 维护，将响应数据修改为统一格式

```
...

return next.handle().pipe(
  map<any, IResponse>(
    value => ({
      errcode: 0,
      errmsg: '',
      data: value,
    }),
  ),
);

...
```

[nestjs 文档 / 拦截器](https://docs.nestjs.cn/5.0/interceptors?id=%e5%9f%ba%e7%a1%80)

### 异常捕获

由 _src/provider/filter/all.exception.filter.ts_ 维护，在 _src/app.module.ts_ 添加为全局注册

- 如果是 gRPC 错误，则需要特殊处理 (稍后在写)
- 如果是 `HttpException`，则返回对应的 `HttpStatus`的状态码，否则返回`HttpStatus.INTERNAL_SERVER_ERROR`状态码

将响应数据修改为统一格式

```
...

response.status(200)
  .json({
    errmsg: message,
    errorCode: status,
  });
```

[nestjs 文档 / 异常过滤器](https://docs.nestjs.cn/5.0/exceptionfilters?id=http-exceptions)

## 创建 HTTP 服务 及 gRPC 服务

### 创建 protobuf

```
// ------ protocol/test/test.proto --------
syntax = "proto3";
package test;

service RpcTestService {
    rpc findOne (testId) returns (test);
}
message testId {
    int32 id = 1;
}
message test {
    string name = 1;
    int32 id = 2;
}
```

### 转换 protobuf

由 _protocol.js_ 维护，使用 **protobufjs** 包，将*.proto 文件转换为*.d.ts 文件，可以在 typescript 中直接使用

```
$ node protocol.js // 方法1
$ npm run build:protocol // 方法2
```

tips: _protocol.js_ 是放在项目根目录的脚本文件

[github/protobuf](https://github.com/protobufjs/protobuf.js)

### 启动混合服务

同时启动 HTTP 服务和 gRPC 服务

```
// ------ src/main.ts ------
...

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: false,
  });

  ...

  app.connectMicroservice(getGRPCClientOption({ package: 'test', protoPath: 'test/service.proto' }));

  app.useLogger(app.get('LogService'));

  app.setGlobalPrefix('/api');

  await app.startAllMicroservicesAsync(); // 启动微服务
  await app.listen(process.env.NODE_ENV === 'production' ? 80 : 7001); // 启动HTTP服务
}

bootstrap();
```

tips: `getGRPCClientOption` 是 _src/utils/rpc.util.ts_ 提供的方法，用于生成 ClientOptions

[nestjs 文档 / 混合服务](https://docs.nestjs.cn/5.0/faq?id=%e6%b7%b7%e5%90%88%e5%ba%94%e7%94%a8)

### 创建 Controller

```
// ------- src/core/controller/test.controller.ts ------

...

@Controller('test')
export class TestController {
  constructor(private readonly testService: TestService) {
  }

  // --------- http method ---------
  @Get()
  // 自动按照 TestDto 规定的格式来校验及转换参数
  index(@Query() query: TestDto): Observable<Itest> {
    return this.testService.findOne(query);
  }

  // --------- gRPC method ---------
  @GrpcMethod('RpcTestService', 'findOne')
  findOne(data: ItestId): Itest {
    return { name: 'test',id: 1 };
  }
}
```

[nestjs 文档 / 控制器](https://docs.nestjs.cn/5.0/controllers)

### 创建 Service

```
// ------- src/core/service/test.service.ts ------

...

@Injectable()
export class TestService {
  // --------- 初始化gRPC客户端、及其他注入 ---------
  private test;

  @Client(getGRPCClientOption({ url: SERVICE_ENUM.TEST_SERVICE, package: 'test', protoPath: 'test/service.proto' }))
  private readonly client: ClientGrpc;

  onModuleInit() {
    this.test = this.client.getService<RpcTestService>('RpcTestService');
  }

  // --------- 业务 ---------

  // 测试代码，gRPC调用
  findOne(query: ItestId): Observable<Itest> {
    return this.test.findOne(query);
  }
}
```

[nestjs 文档 / 提供者](https://docs.nestjs.cn/5.0/providers)

### 创建 Module

创建 _test.module.ts_,并在 _app.module.ts_ 添加该模块

```
// ------- src/core/service/test.module.ts ------

...

@Module({
  controllers: [ TestController ],
  providers: [ TestService ],
})
export class TestModule {}
```

[nestjs 文档 / 模块](https://docs.nestjs.cn/5.0/modules)

### 运行项目

```
$ npm run start:dev
```

项目启动后，可以在浏览器输入 "localhost:7001/api/test" 访问当前接口，此时，会出现 **400**错误，参数校验失败，因为 `id` 是必校验参数。输入 "localhost:7001/api/test?id=2" 则可以得到正常数据

## 部署

### 非容器化部署

```
$ npm run build
$ npm run startup // 使用 pm2 启动
```

### 容器化部署

```
$ npm run build:docker
$ npm run startup:docker
```
