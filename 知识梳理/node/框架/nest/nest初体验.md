# Nest 初体验

## 初始化项目

### 使用 Nest CLI 构建项目

```
$ npm i -g @nestjs/cli
$ nest new nest-example
```

### 项目目录拆分

![](https://user-gold-cdn.xitu.io/2019/9/1/16cecc3c390e52d3?w=288&h=559&f=png&s=7634)

## 基础配置

### 环境变量

引入 **dotenv** 包，读取项目根目录 _.env_ 文件

- _.env_ 文件不提交到 git 仓库，仓库维护 _.env-example_ 文件，在开发环境下由开发人员重命名为 _.env_ 文件，读取开发配置
- 测试、正式环境的 _.env_ 文件由配置仓库统一维护，在启动工程前自动向配置仓库拉取，放置于项目根目录下

[github.com/dotenv](https://github.com/motdotla/dotenv)

### 配置文件

由 _src/provider/service/config.service.ts_ 维护，由 _src/provider/module/config.module.ts_ 添加到 _app.module.ts_

```
//------- config.service.ts -------
...
import defaultConfig from '../../config';
const config: any = Object.assign(
  defaultConfig,
  dotenv.parse(fs.readFileSync(path.join(process.cwd(), '.env'))),
);

@Injectable()
export class ConfigService {
  get(key: string) {
    return config[ key ];
  }
  static get(key: string) {
    return config[ key ];
  }
}
```

```
//------- app.module.ts -------
...
@Module({
  imports: [
    ConfigModule,
  ],
  exports: [
    ConfigModule,
  ],
})
...
```

- 读取 _src/config.ts_ 的配置
- 合并 _src/config.ts_ 的配置及 _.env_ 的配置，属性重名时，以 _.env_ 覆盖 _src/config.ts_
- 提供静态 `get` 方法及实例 `get` 方法，用于获取配置
- 注册为共享模块，可以按照如下示例使用

```
...
@Injectable()
export TestService {
  constructor(private readonly config: ConfigService){
    this.config.get('XX');
    // 或者
    ConfigService.get('XX');
  }
}
```

### 日志

引入 **winston** 包，由 _src/provider/service/log.service.ts_ 维护，由 _src/provider/module/log.module.ts_ 添加到 _app.module.ts_

```
//------- app.module.ts -------
...
@Module({
  imports: [
    LogModule.register({ LOG_PATH: ConfigService.get('LOG_PATH') }),
  ],
  exports: [
    LogModule,
  ],
})
...
```

- 日志路径由 `LOG_PATH` 环境变量维护，由 _config.ts_ 、 _.env_ 或 _Dockerfile_ 文件指定
- 日志切割：日志按天切割，日志格式为 \`YYYY-MM-DD HH:mm:ss [${level}] \${error_message}\`
- 注册为共享模块，可以使用注入来使用

```
...
@Injectable()
export TestService {
  constructor(private readonly logger: LogService){
    this.logger.log('XX');
  }
}
```

tips: 后续准备将日志模块封装为 npm 包，所以目前采用了动态模块加载的方式，不在 _log.service.ts_ 内直接使用项目内的数据

[nestjs 文档 / 日志](https://docs.nestjs.cn/5.0/techniques?id=%e6%97%a5%e5%bf%97)  
[github.com/winston](https://github.com/winstonjs/winston)  
[github.com/winston-daily-rotate-file](https://github.com/winstonjs/winston-daily-rotate-file)

### 登录校验

引入 **@nestjs/jwt** 包，由 _src/provider/guards/auth.guards.ts_ 维护

```
//------- app.module.ts -------
...
@Module({
  imports: [
    JwtModule.register({
      secret: ConfigService.get('JWT').SECRET,
      signOptions: {
        expiresIn: ConfigService.get('JWT').EXPIRES,
      },
    }),
  ],
  exports: [
    JwtModule,
  ],
})
...
```

- 不是每个模块都需要校验登录，所以 _auth.guards.ts_ 未在 _app.module.ts_ 设置，需要在模块中手动设置（如下示例所示），如果你所有模块都需要登录校验，那么可以在 _app.module.ts_ 设置
- 如果未登录或登录失效，则返回 `HttpStatus.UNAUTHORIZED`
- 如果是 gRPC 服务，则需要抛出 `RpcException`，而非抛出 `HttpException`

```
//------- src/core/module/test.module.ts -------
...
@Controller('test')
@UseGuards(AuthGuards)
export class TestController {
}
```

[nestjs 文档 / 授权看守卫](https://docs.nestjs.cn/5.0/guards?id=%e6%8e%88%e6%9d%83%e7%9c%8b%e5%ae%88%e5%8d%ab)

### 请求参数校验

由 nest 的 `ValidationPipe` 维护，在 _src/app.module.ts_ 设置为所有模块生效

```
//------- app.module.ts -------
...
@Module({
    {
      provide: APP_PIPE,
      useValue: new ValidationPipe({ transform: true, validationError: { value: false, target: false } }),
    },
})
...
```

具体参数校验规则由 _src/core/dto/\*.dto.ts_ 各自维护，Demo 如下

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

由 _src/provider/interceptor/response.interceptor.ts_ 维护，将响应数据修改为统一格式，在 _src/app.module.ts_ 设置为所有模块生效

```
//------- response.interceptor.ts -------
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

```
//------- app.module.ts -------
...
@Module({
  {
      provide: APP_INTERCEPTOR,
      useClass: ResponseInterceptor,
    },
})
...
```

[nestjs 文档 / 拦截器](https://docs.nestjs.cn/5.0/interceptors?id=%e5%9f%ba%e7%a1%80)

### 异常捕获

由 _src/provider/filter/all.exception.filter.ts_ 维护，在 _src/app.module.ts_ 设置为所有模块生效

- 如果是 gRPC 错误，则需要特殊处理 (稍后在写)
- 如果是 `HttpException`，则返回对应的 `HttpStatus`的状态码，否则返 `HttpStatus.INTERNAL_SERVER_ERROR`状态码

将响应数据修改为统一格式

```
//------- all.exception.filter.ts -------
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

使用 **protobufjs** 包，由 _protocol.js_ 维护，将*.proto 文件转换为*.d.ts 文件，可以在 typescript 中直接使用

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
  @Client(getGRPCClientOption({ url: ConfigService.get('TEST_SERVICE'), package: 'test', protoPath: 'test/service.proto' }))
  private readonly client: ClientGrpc;

  onModuleInit() {
    this.test = this.client.getService<RpcTestService>('RpcTestService');
  }

  // 测试代码，gRPC调用
  findOne(query: ItestId): Observable<Itest> {
    return this.test.findOne(query);
  }
}
```

[nestjs 文档 / 提供者](https://docs.nestjs.cn/5.0/providers)

### 创建 Module

创建 _test.module.ts_，并在 _app.module.ts_ 添加该模块

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
$ npm run dev
```

项目启动后，进行如下操作：

- 浏览器输入 "localhost:7001/api/test" 访问接口，此时会返回 401，用户处于未登录状态
- 浏览器输入 "localhost:7001/api/user" 访问接口，此时会返回 jwt token
- 浏览器输入 "localhost:7001/api/test?token=\${token}" 访问接口，此时会返回 400,参数校验未通过
- 浏览器输入 "localhost:7001/api/test?token=\${token}&id=1" 访问接口，返回正确结果

## 部署

### 非容器化部署

```
$ npm run build
$ npm run startup // 使用 pm2 启动
```

### 容器化部署

Dockerfile 、 .dockerignore 存放于项目根目录

```
$ npm run build:docker  // 构建 docker
$ npm run startup:docker // docker 内部启动脚本
```

## 项目地址

[nest-example](https://github.com/yuezm/nest-example)
