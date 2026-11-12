# 技术栈插件：Python Django

> 这个插件包含用Django做项目时必须遵守的代码规范和最佳实践。
> 用户选了Django技术栈，AI就必须遵守这里面的所有规则。

---

## 1. 版本要求

> 必须用这些版本或更新的，不然会有兼容问题。

| 依赖 | 最低版本 | 说明 |
|---|---|---|
| Python | 3.10+ | |
| Django | 4.2 LTS | 用LTS长期支持版 |
| Django REST Framework | 3.15+ | 做API接口用 |

---

## 2. 项目结构规范

> Django项目必须这么组织。

```
项目根目录/
├── config/               # 项目配置
│   ├── settings/
│   │   ├── base.py        # 公共配置
│   │   ├── development.py # 开发环境配置
│   │   └── production.py  # 生产环境配置
│   ├── urls.py            # 总路由
│   ├── asgi.py
│   └── wsgi.py
├── apps/                  # 业务模块
│   ├── users/            # 用户模块
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   └── services.py   # 业务逻辑
│   └── products/         # 商品模块
├── static/               # 全局静态文件
├── templates/             # 全局模板
├── media/                 # 用户上传文件
├── manage.py
└── requirements.txt
```

**规则：**
- 每个功能模块放一个app里，不要把所有东西塞一个app
- 配置分dev/prod，不要所有配置塞settings.py
- 业务逻辑写在services.py里，不要全写在views里

---

## 3. 代码生成规则

> AI写Django代码时必须遵守的规则。

### 3.1 模型
- 金额必须用DecimalField，不能用FloatField
- 时间字段自动加created_at和updated_at
- 经常查的字段加db_index=True
- 多用户系统必须有user外键，所有查询加user过滤

### 3.2 接口（DRF）
- 用ViewSet或者APIView，不要用原始Django视图
- 入参出参用Serializer，不要自己手动处理
- 权限必须加IsAuthenticated，不能裸奔
- 列表必须分页，不能一次返回全部

### 3.3 配置
- SECRET_KEY不能硬编码，从环境变量读
- DEBUG在生产必须是False
- ALLOWED_HOSTS生产要配具体域名，不能写*

---

## 4. 常见坑清单

> Django最容易踩的坑。
>
> **清单边界：** 这里只收录已经踩过、确认过的经验，不可能穷尽所有坑。清单没覆盖到的隐性暗坑，第7步开工前AI必须主动搜索该技术栈 + 该场景的官方最佳实践和踩坑记录，补充后再写代码。每做一次项目，踩到新坑就补进这张表。

| 坑 | 为什么错 | 正确做法 |
|---|---|---|
| settings.py里写死SECRET_KEY | 泄露了就完了 | 从环境变量读 |
| DEBUG生产没关 | 暴露敏感信息 | 生产DEBUG=False |
| 所有配置塞一个settings.py | 环境切换麻烦 | 分base/dev/prod |
| 用FloatField存金额 | 精度丢失 | 用DecimalField |
| 没做迁移就改模型 | 数据库和代码不一致 | 每次改完模型makemigrations |
| 做中文RAG用Chroma默认英文embedding模型 | 检索退化成关键词匹配，换个说法就找不到，大模型幻觉 | 换成中文embedding模型（如bge-m3），用Ollama本地跑 |
| 没考虑扫描版/图片PDF | pypdf提取不到文字，文件处理失败 | 主动提示用户：扫描版PDF需要OCR，MVP先不支持 |
| 命令行开全局代理后，本地Ollama/Chroma连接失败 | 本地服务也走代理，502错误 | 必须加NO_PROXY=localhost,127.0.0.1 |
| 国内首次跑Chroma下载默认embedding模型极慢 | HuggingFace直连只有10KB/s | 设HF_ENDPOINT=https://hf-mirror.com，或用Ollama的embedding模型 |
| Ollama默认模型存C盘 | 大模型几十GB，C盘容易满 | 设环境变量OLLAMA_MODELS改到D盘 |
| 做RAG换了embedding模型没重建向量库 | 新旧模型向量不兼容，检索直接乱掉 | 换模型/改chunk大小必须清空旧向量库重新导入 |
| 文本按固定字符数硬切块 | 切到句子中间，语义断裂，检索不准 | 按段落/章节等语义边界切，保留50-100字重叠 |
| 向量库里混了不同模型的向量 | 检索相似度算错，结果不准 | 入库时把embedding模型名写进元数据，检索时按模型过滤 |

---

## 5. 测试规范

> Django怎么写测试。

- 测试写在每个app的tests.py里，或者tests/目录
- 用Django自带的TestCase，不用自己搭
- 测试用独立的测试数据库，跑完自动删
- API测试用APIClient

---

## 6. 部署规范

> Django怎么上线。

- 生产用gunicorn + uwsgi
- 必须跑collectstatic收集静态文件
- 静态文件用Nginx托管，不要让Django处理
- 数据库用PostgreSQL，不要用SQLite
- 必须配HTTPS

---

## 7. 反模式清单

> AI最容易犯的错误。

| 反模式 | 为什么错 | 怎么改 |
|---|---|---|
| 所有东西写在一个views.py里 | 项目大了没法维护 | 按功能分app |
| 业务逻辑写在views里 | 没法复用，不好测 | 抽到services里 |
| 用原始SQL拼接 | SQL注入风险 | 用ORM或参数化查询 |
| 没做权限校验 | 越权访问 | 加permission_classes |

---

## 来源

> 这个插件的内容来自哪里。

- Django官方文档
- Django项目结构最佳实践
