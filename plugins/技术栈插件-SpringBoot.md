# 技术栈插件：Java Spring Boot

> 这个插件包含用Spring Boot做项目时必须遵守的代码规范和最佳实践。
> 用户选了Spring Boot技术栈，AI就必须遵守这里面的所有规则。

---

## 1. 版本要求

> 必须用这些版本或更新的，不然会有兼容问题。

| 依赖 | 最低版本 | 说明 |
|---|---|---|
| Java | 17 LTS | 用LTS长期支持版 |
| Spring Boot | 3.2.x | |
| Maven | 3.8+ | 构建工具 |

---

## 2. 项目结构规范

> Spring Boot项目推荐按业务功能分包，不要按技术层分包。

```
src/main/java/com/你的项目名/
├── Application.java          # 入口类
├── config/                  # 配置类
│   ├── SecurityConfig.java   # 安全配置
│   └── CorsConfig.java       # 跨域配置
├── common/                  # 公共模块
│   ├── result/              # 统一返回格式
│   ├── exception/          # 全局异常处理
│   └── utils/              # 工具类
└── modules/                 # 业务模块（按功能分）
    ├── user/                # 用户模块
    │   ├── controller/      # 接口
    │   ├── service/         # 业务逻辑
    │   ├── mapper/          # 数据访问
    │   ├── entity/          # 数据库实体
    │   └── dto/             # 入参出参
    └── product/             # 商品模块
```

**规则：**
- 业务逻辑必须写在Service层，不能写在Controller里
- 数据库操作必须通过Mapper/Repository，不能在Controller里写SQL
- 一个业务模块一个包，不要把所有东西塞一起
- Entity和DTO分开，不能直接把数据库实体返回给前端

---

## 3. 代码生成规则

> AI写Spring Boot代码时必须遵守的规则。

### 3.1 分层
- Controller：只负责接请求、调Service、返回结果
- Service：业务逻辑，一个方法一个功能
- Mapper：数据库操作，用MyBatis-Plus或者JPA
- Entity：数据库表对应的实体类
- DTO：接口入参出参，不能直接用Entity

### 3.2 数据库
- 金额必须用BigDecimal，不能用Float/Double
- 时间字段用LocalDateTime，不要用Date
- 经常查的字段加索引
- 多用户系统所有表必须有user_id，所有查询加用户过滤

### 3.3 接口
- 统一返回格式：{code, message, data}
- 必须加参数校验（@Valid）
- 所有业务接口必须加登录校验，不能裸奔
- 列表接口必须分页

### 3.4 异常处理
- 全局异常处理器，统一返回错误格式
- 不要在Controller里try-catch吞异常
- 业务异常抛自定义异常类

---

## 4. 常见坑清单

> Spring Boot最容易踩的坑。
>
> **清单边界：** 这里只收录已经踩过、确认过的经验，不可能穷尽所有坑。清单没覆盖到的隐性暗坑，第7步开工前AI必须主动搜索该技术栈 + 该场景的官方最佳实践和踩坑记录，补充后再写代码。每做一次项目，踩到新坑就补进这张表。

| 坑 | 为什么错 | 正确做法 |
|---|---|---|
| 用Float存金额 | 精度丢失 | 用BigDecimal |
| 把Entity直接返回给前端 | 暴露敏感字段 | 用DTO |
| 业务逻辑写在Controller里 | 没法复用，不好测 | 抽到Service里 |
| 数据库密码硬编码在application.yml | 不安全 | 从环境变量读 |
| 生产开actuator端点 | 泄露敏感信息 | 加权限控制或者关掉 |

---

## 5. 测试规范

> Spring Boot怎么写测试。

- 用Spring Boot Test + JUnit 5
- Controller测试用MockMvc
- Service测试用Mockito mock依赖
- 数据库测试用H2内存库或者Testcontainers
- 每个测试方法只测一个场景

---

## 6. 部署规范

> Spring Boot怎么上线。

- 打包成jar包，用java -jar跑
- 生产用外部配置，不要把配置打在包里
- 必须配HTTPS
- 数据库连接池用HikariCP（默认就是）

---

## 7. 反模式清单

> AI最容易犯的错误。

| 反模式 | 为什么错 | 怎么改 |
|---|---|---|
| 所有东西塞一个Controller | 项目大了没法维护 | 按功能分模块 |
| Controller里写业务逻辑 | 分层违规 | 抽到Service |
| 直接返回Entity不设DTO | 暴露敏感字段 | 加DTO转换 |
| SQL拼接字符串 | SQL注入 | 用MyBatis参数化或者JPA |
| 全局异常没处理 | 报错返回乱七八糟 | 加@RestControllerAdvice |

---

## 来源

> 这个插件的内容来自哪里。

- Spring Boot官方文档
- Spring Boot最佳实践指南
