# 🧩 Prompt Mastery Kit

> 与大模型交互的工程化模板套件 | 一纸速查风格 | 兼容 Claude / MiniMax

------

## 📦 套件结构

```
prompt-mastery-kit/
├── framework/
│   ├── framework_onboarding.md   # 开源框架快速上手 · 速查文档
│   └── framework.prompt          # 开源框架 Prompt 模板
├── legacy/
│   ├── legacy_system_onboarding.md  # 遗留业务系统解析 · 速查文档
│   └── legacy.prompt                # 遗留系统 Prompt 模板
└── README.md                     # 本文件
```

------

## 🗺️ 使用路径速查

```
你的场景
  │
  ├─ 学一个新框架？
  │     └─► framework_onboarding.md  →  填写 framework.prompt  →  投喂模型
  │
  └─ 接手遗留系统？
        └─► legacy_system_onboarding.md  →  收集输入物料  →  填写 legacy.prompt  →  投喂模型
```

------

## ⚡ 30秒上手规则

| 步骤 | 动作                                                         |
| ---- | ------------------------------------------------------------ |
| 1    | 打开对应场景的 `.md` 速查文档，确认你处于哪个阶段            |
| 2    | 打开对应 `.prompt` 文件，替换所有 `{{占位符}}`               |
| 3    | 选择模型变体：**Claude** 用 `<xml>` 风格 / **MiniMax** 用角色扮演风格 |
| 4    | 粘贴进模型，按文档指引追问                                   |

------

## 🔑 占位符规范

所有模板统一使用 `{{variable_name}}` 格式，对齐 LangChain / PromptFlow 生态。

| 占位符类型 | 示例                 | 说明                              |
| ---------- | -------------------- | --------------------------------- |
| 单值文本   | `{{framework_name}}` | 直接替换为字符串                  |
| 多行内容   | `{{git_log}}`        | 替换为完整多行文本块              |
| 可选项     | `{{pain_point?}}`    | 末尾 `?` 表示非必填，无则删除整行 |

------

## 🤖 模型选型建议

| 场景             | 推荐模型 | 原因                           |
| ---------------- | -------- | ------------------------------ |
| 源码/架构分析    | Claude   | 长上下文 + 结构化推理更强      |
| 业务语义摘要     | MiniMax  | 中文业务理解流畅，角色扮演稳定 |
| Git Log 变更分析 | Claude   | 代码 diff 解析精度高           |
| 口头描述转文档   | MiniMax  | 自然语言归纳能力强             |

------

> 📌 模板版本 v1.0 · 适用模型：Claude Sonnet / MiniMax abab6.5