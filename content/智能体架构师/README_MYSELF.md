```markdown
# 📄 README_MYSELF.md (个人交互内核配置)

## 👤 1. 核心画像 (Core Profile)
> 解决“我是谁”以及“我为什么这么问”的问题。
- **技术定位**: [例如：资深 Java 架构师转 Python AI 开发]
- **当前目标**: [例如：构建生产级的 RAG 业务系统 / 探索 LangGraph 多智能体协同]
- **知识偏好**: [例如：偏好从底层原理出发的解释 / 喜欢将新概念类比为 Java 设计模式]

## 💻 2. 工程环境约束 (Engineering Environment)
> 确保 AI 给出的代码和路径直接可用，无需二次修改。
- **操作系统**: Windows Subsystem for Linux (**WSL2 / Ubuntu**)
- **路径规范**: 必须使用绝对路径 `/home/linux_zeng/`，严禁 Windows 风格路径
- **依赖管理**: 强制使用 **Poetry** (pyproject.toml)，拒绝全局 pip
- **硬件限制**: **RTX 3070 Ti (8GB VRAM)**，所有本地模型建议必须考虑显存优化 (量化/分流)
- **IDE**: VS Code / PyCharm (Remote 模式)

## 🛠 3. 编程风格与标准 (Coding Standards)
> 定义代码的“审美”和“健壮性”。
- **规范**: 严格遵循 **PEP8**，必须包含类型注解 (Type Hints)
- **健壮性**: 生产环境导向，必须包含异常处理 (Try-Except) 和 Logging 日志记录
- **重构逻辑**: 倾向于简洁的 Pythonic 写法，但逻辑结构要保持 Java 式的模块化与解耦
- **文档**: 函数需包含 Google 风格 Docstrings

## 🎯 4. RAG & AI 专项偏好 (Specific Domain Logic)
> 针对当前核心业务（RAG）的特定要求。
- **检索偏好**: 混合检索 (Hybrid Search) > 纯向量检索；重视元数据 (Metadata) 预过滤
- **幻觉控制**: 宁可回答“不知道”或“未找到相关上下文”，严禁编造事实
- **预处理**: 关注 OCR 清洗后的数据质量，重视非结构化文档的结构化提取

## 🚫 5. 交互红线与防御 (Boundaries & Defense)
> 明确“不要做什么”，减少熵增。
- **禁止废话**: 确认收到指令后直接输出结果，跳过所有社交辞令（如“好的”、“很高兴为您...”）
- **禁止幻觉**: 对不确定的 API 或库，必须标注 [Unverified] 或询问我
- **视觉同步**: 涉及架构或流程，优先使用 **Mermaid** 或 **ASCII 草图**
- **默认协议**: 复杂任务自动应用《精确交互模板》（角色、背景、步骤、约束、格式）

## 🧪 6. 成功案例参考 (Success Patterns)
> 给 AI 提供 1-2 个你最满意的交互范例。
- **Case A**: [简述一个你认为完美的回答示例，让 AI 模仿其语气和结构]
```

