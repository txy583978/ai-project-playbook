# 技术栈插件：Python FastAPI

> 这个插件包含用FastAPI做项目时必须遵守的代码规范和最佳实践。
> 用户选了FastAPI技术栈，AI就必须遵守这里面的所有规则。

---

## 1. 版本要求

> 必须用这些版本或更新的，不然会有兼容问题。

| 依赖 | 最低版本 | 说明 |
|---|---|---|
| Python | 3.10 ~ 3.11 | 别用太新的版本，有些包没预编译wheel |
| FastAPI | 0.115 | |
| Pydantic | 2.7 | 用v2，别用v1的写法 |
| pydantic-settings | 2.4 | |
| SQLAlchemy | 2.0 | |
| httpx | 0.27 | 测试用 |
| passlib | 1.7.4 | 密码哈希 |
| bcrypt | 4.0.1 | 锁死版本，不然passlib会报错 |

---

## 2. 项目结构规范

> 目录必须这么分，不能乱建文件。

```
项目根目录/
├── core/               # 核心配置
│   ├── config.py       # 配置管理（从.env读）
│   ├── security.py     # JWT、密码哈希
│   ├── deps.py         # 依赖注入（获取当前用户）
│   ├── response.py     # 统一响应格式
│   └── exceptions.py   # 全局异常处理
├── db/                 # 数据库
│   └── database.py     # engine、session、Base
├── models/             # ORM模型
│   └── user.py        # 一个表一个文件
├── schemas/            # Pydantic入参出参
│   └── user.py
├── routers/            # 接口路由
│   └── products.py    # 一个功能一个文件
├── services/           # 业务逻辑
│   └── product_service.py
├── tests/              # 测试
│   ├── conftest.py     # pytest配置
│   └── api/
├── scripts/            # 脚本（初始化数据等）
├── main.py             # 入口文件
├── requirements.txt    # 依赖
├── .env.example        # 环境变量示例
└── .gitignore
```

**规则：**
- 业务逻辑必须写在services里，不能写在routers里
- 数据库模型放models，Pydantic入参出参放schemas
- 一个功能一个文件，不要把所有东西塞一个文件里
- routers只负责接请求、调service、返回结果

---

## 3. 代码生成规则

> AI写代码时必须遵守的规则。

### 3.1 配置管理
- 所有配置都从环境变量读，不能硬编码
- config.py 给开发默认值，没有 .env 也能启动；但 prod 环境下仍取到兜底默认值必须拒绝启动（见坑表护栏条）
- 用pydantic-settings的BaseSettings，不要自己读os.environ

### 3.2 数据库
- 金额必须用DECIMAL(Numeric)，不能用FLOAT——不然对账差几分钱
- 时间字段必须带时区（DateTime(timezone=True)）
- 经常查的字段必须加索引
- 多用户系统所有业务表必须有user_id字段
- 所有业务查询必须加WHERE user_id = 当前登录用户，做越权检查

### 3.3 接口
- 所有接口都要加response_model，不能直接返回ORM对象——不然会暴露敏感字段
- 入参必须用Pydantic模型，加Field约束：
  - 字符串：min_length、max_length
  - 数字：ge=0（价格、库存不能负数）
  - 邮箱/手机号：用专门的校验类型
- 统一返回格式：{code: 0, message: "success", data: ...}
- 列表接口必须分页，不能一次返回全部数据

### 3.4 鉴权
- 登录用JWT，密码用bcrypt哈希，不能明文存
- 获取当前用户用Depends依赖注入，不能在每个接口里自己写
- 所有业务接口必须加get_current_user_id依赖，不能裸奔

### 3.5 async/sync选择
- 如果接口里都是非阻塞I/O（数据库查询、HTTP请求），用async def
- 如果用的是同步库（比如老的requests），用普通def，FastAPI会自动放线程池里跑
- 不要在async路由里调同步库——会阻塞整个事件循环

---

## 4. 常见坑清单

> FastAPI最容易踩的坑，AI必须主动避开。
>
> **清单边界：** 这里只收录已经踩过、确认过的经验，不可能穷尽所有坑。清单没覆盖到的隐性暗坑，第7步开工前AI必须主动搜索该技术栈 + 该场景的官方最佳实践和踩坑记录，补充后再写代码。每做一次项目，踩到新坑就补进这张表。

| 坑 | 为什么错 | 正确做法 |
|---|---|---|
| 测试import main时连真数据库 | 环境没配好就崩 | 测试前先在import main之前设环境变量为SQLite |
| 临时改dependency_overrides不恢复 | 断言失败后面所有测试全污染 | 用try-finally恢复 |
| bcrypt装最新版 | passlib 1.7.4兼容报错 | requirements里锁死bcrypt==4.0.1 |
| Python用最新版本 | pydantic-core要现场编译，失败 | 用3.10~3.11 |
| config.py不给默认值 | 没有.env就启动报错 | 所有配置项给开发默认值 |
| dev 兜底默认值带进 prod | 部署漏配也不报错，默认密钥 / 白名单直接上线 | prod/production 下仍取兜底默认值时启动即拒绝（非 0 退出）；未配置与取到默认值两种都判不安全；密钥、数据库密码、CORS 白名单必查；配 1 条拒绝启动用例 |
| main.py顶层create_all | 测试import时就连真库 | 把建表放到lifespan里 |
| 写死user_id=1但没插测试用户 | 外键约束报错 | 测试fixture里先插对应id的用户 |
| .dict()用在Pydantic v2 | 已废弃，有警告 | 改成.model_dump() |
| main.py末尾没有启动块 | python main.py跑不起来 | 加if __name__ == "__main__" |
| `/docs` 白屏，但 `/openapi.json` 能正常返回 | FastAPI 默认 Swagger UI 的 CSS/JS 从 jsdelivr CDN 加载，网络受限/离线时脚本加载失败，页面只剩空 div；后端接口本身没问题 | 先请求 `/openapi.json`（200 且有 JSON）区分是后端问题还是静态资源问题；CDN 不可达时把 Swagger 资源换到可达镜像，或自托管 swagger-ui-dist 文件；调试也可直接用 Apifox 或前端联调，不依赖 `/docs` |

---

## 5. 测试规范

> FastAPI怎么写测试。

### 5.1 测试隔离
- 必须用SQLite内存库，不能连真数据库
- 必须在import main之前设置环境变量，强制DATABASE_URL为sqlite
- 鉴权必须override，不能真的发请求登录
- 每个测试跑完清库，不能互相影响

### 5.2 标准fixture写法（完整可运行）
```python
import os
# 必须在import main之前设环境变量，强制用SQLite内存库
os.environ["DATABASE_URL"] = "sqlite:///:memory:"

import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from main import app
from db.database import Base, get_db
from core.deps import get_current_user_id
from models.user import User

# 创建测试用的engine和session
test_engine = create_engine("sqlite:///:memory:")
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=test_engine)

# override数据库依赖
def override_get_db():
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

# override鉴权依赖
def override_get_current_user_id():
    return 1  # 测试用户id=1

app.dependency_overrides[get_db] = override_get_db
app.dependency_overrides[get_current_user_id] = override_get_current_user_id

@pytest.fixture
def client():
    # 建表
    Base.metadata.create_all(bind=test_engine)
    
    # 插入测试用户（id=1）
    db = TestingSessionLocal()
    test_user = User(
        id=1, 
        username="testuser", 
        phone="13800138000",
        password_hash="$2b$12$fakehashfortest"
    )
    db.add(test_user)
    db.commit()
    db.close()
    
    # 返回测试客户端
    yield TestClient(app)
    
    # 清理：删表
    Base.metadata.drop_all(bind=test_engine)
```

### 5.3 测试用例
- 必须测正常场景（200成功）
- 必须测异常场景（401未登录、404不存在、422参数错）
- 越权场景必须测：用户A查用户B的数据应该404

---

## 6. 部署规范

> FastAPI怎么上线。

- 开发用`uvicorn main:app --reload`
- 生产用`gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app`
- 必须用HTTPS
- 数据库密码不能写在代码里，必须从环境变量读
- 开发用SQLite，生产换MySQL/PostgreSQL

---

## 7. 反模式清单

> AI最容易犯的错误，代码审查时必须检查。

| 反模式 | 为什么错 | 怎么改 |
|---|---|---|
| 直接返回ORM对象不设response_model | 会暴露密码hash等敏感字段 | 加response_model参数 |
| async路由里用同步requests库 | 阻塞事件循环 | 用httpx.AsyncClient或run_in_threadpool |
| 业务逻辑写在router里 | 分层违规 | 抽到service里 |
| 路由里catch Exception吞错误 | 把500变成200，出问题找不到 | 抛HTTPException带正确状态码 |
| Field(ge=18, default=None) | 约束和默认值矛盾 | 要么必填要么可选，别写一起 |
| 所有配置塞一个BaseSettings | 项目大了难维护 | 按模块分配置 |
| text()/原生SQL拼接用户输入 | SQL注入 | 用ORM参数化查询，不要拼字符串 |
| os.system/subprocess 用shell=True拼接用户输入 | 命令注入 | 用参数列表形式调用，不走shell |
| 用户输入拼文件路径open() | 路径穿越，读到目录外文件 | 限制在固定目录内，校验文件名 |
| pickle / yaml.load 反序列化不可信数据 | 反序列化漏洞 | 不用pickle；yaml改用safe_load |
| 业务接口漏挂 JWT 鉴权依赖 | 未登录也能调 | 统一加get_current_user依赖，不裸奔 |

---

## 来源

> 这个插件的内容来自哪里。

- [FastAPI Best Practices for AI Agents](https://github.com/zhanymkanov/fastapi-best-practices)
- 我们自己做示例项目踩的所有坑
- OWASP FastAPI Security Cheat Sheet
