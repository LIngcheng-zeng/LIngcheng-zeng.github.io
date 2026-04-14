



```sh
# lunix/ubantu 安装ollama
curl -fsSL https://ollama.com/install.sh | sh
# 命令home
/usr/local/lib/ollama

# 下载并手动解压
curl -fsSL https://ollama.com/download/ollama-linux-amd64.tar.zst \ 
| sudo tar x -C /usr

# 启动ollama
ollama serve

# 确认ollama在运行
ollama -v

# 添加ollama作为自启动服务
sudo useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
sudo usermod -a -G ollama $(whoami)
# sudo 以超级用户（root）权限执行
# useradd 创建用户的指令
# -r 创建系统用户，这种用户通常用于运行后台服务，而不是给人登录用的。它的用户 ID (UID) 通常很小（小于 1000）。
# -s /bin/false 禁止登录
# -U 创建同名用户组 ，自动创建一个也叫 ollama 的用户组，并将 ollama 用户加入其中。
# -m 创建home目录
# -d /usr/share/ollama 指定home目录路径

# usermod 修改用户属性的指令
# -a 追加 (Append)
# -G 附加组
# 把你（当前操作电脑的人）加入到 ollama 用户组里

模型默认存储在 /usr/share/ollama/.ollama

# 启动neo4j
sudo service neo4j start
```

