```shell
# 安装 PaddlePaddle（CPU版，适合大多数人）
pip install paddlepaddle

# 安装 PaddleOCR
pip install paddleocr

# 使用ocr
paddleocr ocr -i ./../projects/ocrAndexcel/upload --lang ch --enable_mkldnn False

python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```



```sh
ollama run gemma4:26b
ollama launch openclaw --model qwen3.5
ollama launch openclaw --model gemma4:26b


sudo systemctl daemon-reload
sudo systemctl restart ollama
确认 API 地址是 http://127.0.0.1:11434 还是 http://localhost:11434，两个都试试。
```





```sh
openclaw dashboard

Usage: openclaw [options] [command]

Options:
  --container <name>   Run the CLI inside a running Podman/Docker container named <name> (default: env OPENCLAW_CONTAINER)
  --dev                Dev profile: isolate state under ~/.openclaw-dev, default gateway port 19001, and shift derived ports
                       (browser/canvas)
  -h, --help           Display help for command
  --log-level <level>  Global log level override for file + console (silent|fatal|error|warn|info|debug|trace)
  --no-color           Disable ANSI colors
  --profile <name>     Use a named profile (isolates OPENCLAW_STATE_DIR/OPENCLAW_CONFIG_PATH under ~/.openclaw-<name>)
  -V, --version        output the version number

Commands:
  Hint: commands suffixed with * have subcommands. Run <command> --help for details.
  acp *                Agent Control Protocol tools
  agent                Run one agent turn via the Gateway
  agents *             Manage isolated agents (workspaces, auth, routing)
  approvals *          Manage exec approvals (gateway or node host)
  backup *             Create and verify local backup archives for OpenClaw state
  browser              Manage OpenClaw's dedicated browser (Chrome/Chromium)
  channels *           Manage connected chat channels (Telegram, Discord, etc.)
  clawbot *            Legacy clawbot command aliases
  completion           Generate shell completion script
  config *             Non-interactive config helpers (get/set/unset/file/validate). Default: starts guided setup.
  configure            Interactive configuration for credentials, channels, gateway, and agent defaults
  cron *               Manage cron jobs via the Gateway scheduler
  daemon *             Gateway service (legacy alias)
  dashboard            Open the Control UI with your current token
  devices *            Device pairing + token management
  directory *          Lookup contact and group IDs (self, peers, groups) for supported chat channels
  dns *                DNS helpers for wide-area discovery (Tailscale + CoreDNS)
  docs                 Search the live OpenClaw docs
  doctor               Health checks + quick fixes for the gateway and channels
  gateway *            Run, inspect, and query the WebSocket Gateway
  health               Fetch health from the running gateway
  help                 Display help for command
  hooks *              Manage internal agent hooks
  logs                 Tail gateway file logs via RPC
  mcp                  Manage OpenClaw MCP config and channel bridge
  memory               Search, inspect, and reindex memory files
  message *            Send, read, and manage messages
  models *             Discover, scan, and configure models
  node *               Run and manage the headless node host service
  nodes *              Manage gateway-owned node pairing and node commands
  onboard              Interactive onboarding for gateway, workspace, and skills
  pairing *            Secure DM pairing (approve inbound requests)
  plugins *            Manage OpenClaw plugins and extensions
  qr                   Generate iOS pairing QR/setup code
  reset                Reset local config/state (keeps the CLI installed)
  sandbox *            Manage sandbox containers for agent isolation
  secrets *            Secrets runtime reload controls
  security *           Security tools and local config audits
  sessions *           List stored conversation sessions
  setup                Initialize local config and agent workspace
  skills *             List and inspect available skills
  status               Show channel health and recent session recipients
  system *             System events, heartbeat, and presence
  tasks *              Inspect durable background task state
  tui                  Open a terminal UI connected to the Gateway
  uninstall            Uninstall the gateway service + local data (CLI remains)
  update *             Update OpenClaw and inspect update channel status
  webhooks *           Webhook helpers and integrations
```

