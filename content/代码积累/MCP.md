

❯ mcp 服务器怎么配置，我想把本地启动的 graphity mcp server 配置上

● 两种方式二选一：

---
  方式一：CLI 命令（推荐，自动生成配置）

  claude mcp add --transport stdio graphify --scope project -- \
    python3 -m graphify.serve /home/linux_zeng/projects/classRelation/graphify-out/graph.json

  --scope project 会在项目根目录生成 .mcp.json，只对当前项目生效。

---
  方式二：手动创建 .mcp.json

  在项目根目录创建 .mcp.json：

  {
    "mcpServers": {
      "graphify": {
        "command": "python3",
        "args": ["-m", "graphify.serve", "graphify-out/graph.json"]
      }
    }
  }

  ▎ 注意：路径用相对路径时，工作目录是项目根目录。

---
  验证

  配置后在 Claude Code 中运行：
  /mcp
  看到 graphify 状态为 connected 即生效。也可用：
  claude mcp list

---
## MCP 完整配置如下

```json
{
  "mcpServers": {
    "graphiti": {
      "transport": "stdio",
      "command": "/home/linux_zeng/.local/bin/uv",
      "args": [
        "run",
        "--directory",
        "/home/linux_zeng/projects/graphiti/mcp_server",
        "--project",
        ".",
        "main.py",
        "--transport",
        "stdio"
      ],
      "env": {
        "NEO4J_URI": "bolt://localhost:7687",
        "NEO4J_USER": "neo4j",
        "NEO4J_PASSWORD": "xx",
        "OPENAI_API_KEY": "ollama",
        "OPENAI_API_URL": "http://localhost:11434/v1"
      }
    }
  }
}
```

