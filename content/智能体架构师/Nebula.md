```
  nebula  相关镜像库
  docker run -itd --restart=always  -p 7001:7001  docker.io/vesoft/nebula-graph-studio:v3.8.0
  https://docker.aityp.com/i/search?site=All&platform=All&sort=%E5%90%8D%E7%A7%B0%E6%8E%92%E5%BA%8F&search=nebula-studio
  
  
  cd ~/nebula/nebula-studio
  docker compose up -d

  第三步：访问 Studio

  浏览器打开 http://localhost:7001，连接配置填写：

  ┌──────────┬──────────────────────┐
  │   字段   │          值          │
  ├──────────┼──────────────────────┤
  │ Host     │ host.docker.internal │
  ├──────────┼──────────────────────┤
  │ Port     │ 9669                 │
  ├──────────┼──────────────────────┤
  │ Username │ root                 │
  ├──────────┼──────────────────────┤
  │ Password │ nebula（默认）       │
  └──────────┴──────────────────────┘

---

  关键说明：

  - extra_hosts: host.docker.internal:host-gateway — Linux/WSL2 必须显式声明，否则容器内无法解析该域名
  - Studio 本身无状态，连接信息在 UI 登录页配置，不在 compose 文件中硬编码
```

