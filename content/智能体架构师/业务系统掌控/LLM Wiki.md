````markdown
# 🧩 Skill 设计方案 — LLM Wiki（项目文档知识库）

## 一、核心思想

基于 Karpathy 的 LLM Wiki 模式：**知识编译而非知识检索**。

不同于 RAG 每次从原始文档重新发现答案，LLM Wiki 让 Claude Code
持续维护一个结构化的 Markdown Wiki：
- 新资料进来时主动整合、更新交叉引用、标记矛盾
- 知识随时间复利增长，查询结果可归档回流至 Wiki
- LLM 负责所有维护工作，人只负责资料来源与方向引导

---

## 二、适用场景

- **领域**：Java 项目文档知识库
- **原始资料**：Java 源码、配置文件（`.yml` / `.properties` / `.xml`）、`pom.xml`
- **存储形式**：本地 Markdown 文件

---

## 三、目录结构

### 原始资料（只读，独立目录）

```
/path/to/java-project/
├── src/
│   └── main/
│       ├── java/          # Java 源码
│       └── resources/     # 配置文件
└── pom.xml
```

### Wiki 知识库（独立目录）

```
/path/to/project-wiki/
├── CLAUDE.md              # Skill Schema & 行为规范（Claude Code 入口）
├── .wiki-config           # 记录 raw 源路径的配置文件
└── wiki/
    ├── index.md           # 页面目录（按类别组织）
    ├── log.md             # 操作日志（append-only）
    ├── overview.md        # 项目全貌综述
    ├── entities/          # 实体页（核心类 / 模块）
    ├── concepts/          # 概念页（架构模式 / 设计决策）
    ├── configs/           # 配置解析页
    └── queries/           # 查询结果归档页
```

> wiki 与 raw 完全解耦，`.wiki-config` 作为唯一路径锚点，
> 支持对接多个不同路径的 Java 项目。

---

## 四、三层架构

```
┌──────────────────────────────────────────────────────┐
│                     Raw Sources                       │
│  Java src / @Entity / @Service / application.yml     │
│  mybatis mapper / spring config / pom.xml            │
│  （只读，LLM 读取但不修改）                            │
└─────────────────────────┬────────────────────────────┘
                           ↓  ingest / lint / query
┌──────────────────────────────────────────────────────┐
│                       Wiki                            │
│  entities/  concepts/  configs/  queries/            │
│  index.md   log.md     overview.md                   │
│  （LLM 全权维护，Markdown 落盘）                       │
└─────────────────────────┬────────────────────────────┘
                           ↑  行为约束
┌──────────────────────────────────────────────────────┐
│                    CLAUDE.md Schema                   │
│  页面类型规范 / 操作流程 / 命名约定 / 更新策略          │
└──────────────────────────────────────────────────────┘
```

---

## 五、.wiki-config 规范

```yaml
# Wiki 配置文件，记录原始资料路径
raw_sources:
  - path: /path/to/java-project
    type: maven
    include:
      - src/main/java/**/*.java
      - src/main/resources/**/*.yml
      - src/main/resources/**/*.properties
      - src/main/resources/**/*.xml
      - pom.xml
    exclude:
      - src/test/**

wiki_root: ./wiki
log: ./wiki/log.md
index: ./wiki/index.md
```

---

## 六、Wiki 页面类型规范

| 页面类型 | 存放路径 | 触发条件 | 核心字段 |
|---|---|---|---|
| `ENTITY` | `entities/` | 扫描到 `@Entity` / `@Table` | 类名、表名、字段列表、关联关系 |
| `SERVICE` | `entities/` | 扫描到 `@Service` / `@Component` | 职责描述、依赖注入、核心方法 |
| `CONCEPT` | `concepts/` | 识别到架构模式 / 设计决策 | 问题背景、解决方案、权衡说明 |
| `CONFIG` | `configs/` | 解析 `.yml` / `.properties` / `.xml` | 配置项、默认值、影响模块 |
| `QUERY` | `queries/` | 有价值的查询结果 | 问题、答案、引用页面 |

---

## 七、页面 Frontmatter 规范

```markdown
---
type: ENTITY | SERVICE | CONCEPT | CONFIG | QUERY
source: /path/to/java-project/src/main/java/com/example/UserEntity.java
updated: 2026-04-08
confidence: high | medium | low
tags: [user, jpa, auth]
links: [entities/OrderEntity.md, concepts/AuthFlow.md]
---
```

---

## 八、核心操作指令

```
操作              示例                          说明
─────────────────────────────────────────────────────────────
wiki:init        wiki:init                     初始化 wiki 目录结构
                                               读取 .wiki-config
                                               生成 index.md / log.md

wiki:ingest      wiki:ingest UserEntity.java   按文件名/路径从
                 wiki:ingest src/service/      raw_sources 中定位
                                               并摄入单文件或目录

wiki:ingest-all  wiki:ingest-all               全量扫描所有
                                               raw_sources

wiki:query       wiki:query "用户鉴权流程"      读 index.md → 定位
                                               相关页 → 综合答案
                                               → 归档至 queries/

wiki:lint        wiki:lint                     扫描孤儿页 / 过时内容
                                               / 缺失交叉引用
                                               / 矛盾声明
```

---

## 九、log.md 条目格式

```markdown
## [2026-04-08] ingest | UserEntity.java
- 新建: entities/UserEntity.md
- 更新: entities/OrderEntity.md（补充反向关联）
- 更新: index.md
```

---

## 十、index.md 结构

```markdown
# Wiki Index

## Entities
- [UserEntity](entities/UserEntity.md) — 用户表，关联 Order / Role
- [OrderEntity](entities/OrderEntity.md) — 订单表，含状态机

## Services
- [UserService](entities/UserService.md) — 用户注册、登录、权限校验

## Concepts
- [AuthFlow](concepts/AuthFlow.md) — JWT 鉴权全链路

## Configs
- [ApplicationConfig](configs/ApplicationConfig.md) — 数据源、Redis、线程池

## Queries
- [Q-20260408-AuthFlow](queries/Q-20260408-AuthFlow.md) — 用户鉴权流程完整链路
```

---

## 十一、实现模块规划

```
Step 1 → CLAUDE.md        Skill Schema & 行为规范
Step 2 → wiki:init        初始化脚本
Step 3 → wiki:ingest      摄入流水线（Java 源码 + 配置文件解析）
Step 4 → wiki:query       查询与归档
Step 5 → wiki:lint        健康检查
```
````

