# Windows VS Code Remote-SSH / Dev Container / Codex 配置记录



> - `<WINDOWS_PROXY_IP>`：Windows 主机的局域网或 Tailscale IP
>- `<REMOTE_SERVER_IP>`：Ubuntu 远程服务器的 IP
> - `<REMOTE_HOST>`：Remote-SSH 使用的主机别名
> - `<REMOTE_USER>`：远程服务器用户名

## 1. 环境架构

```text
Windows（<WINDOWS_PROXY_IP>）
├─ Clash Verge 提供代理：<WINDOWS_PROXY_IP>:7897
├─ 本地 VS Code
└─ 通过 Remote-SSH 连接 Ubuntu 服务器

Ubuntu（<REMOTE_HOST>）
├─ 存放并运行项目代码
├─ 运行 VS Code Server
├─ 运行远程 Codex 扩展
└─ 通过 HTTP_PROXY / HTTPS_PROXY 使用 Windows 代理
```

Codex 的运行位置取决于当前 VS Code 窗口：

```text
Windows 本地项目   → Windows 版 Codex
Remote-SSH 项目    → Ubuntu 上的 Linux 版 Codex
Dev Container 项目 → 容器内的 Linux 版 Codex
```

日常使用流程：

1. Windows 启动 Clash Verge。
2. 确保代理端口 `7897` 允许局域网或 Tailscale 设备访问。
3. VS Code 通过 Remote-SSH 连接 Ubuntu 服务器。
4. 打开远程项目，或进一步附加到 Docker 容器。
5. 在对应的远程窗口中使用 Codex。
6. Codex 在远程环境中读取代码、修改文件、运行命令和测试。
7. 网络请求通过 `Ubuntu/容器 → Windows Clash → Internet` 转发。

---

## 2. Remote-SSH + Codex

### 2.1 在 Ubuntu 服务器配置代理

创建统一的代理环境文件：

```bash
cat > ~/.proxy_env <<'EOF'
export HTTP_PROXY="http://<WINDOWS_PROXY_IP>:7897"
export HTTPS_PROXY="http://<WINDOWS_PROXY_IP>:7897"

export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"

export NO_PROXY="localhost,127.0.0.1,::1,<WINDOWS_PROXY_IP>,<REMOTE_SERVER_IP>"
export no_proxy="$NO_PROXY"
EOF
```

让登录 Shell 和交互式 Bash 自动加载：

```bash
grep -qxF '[ -f "$HOME/.proxy_env" ] && . "$HOME/.proxy_env"' ~/.profile \
  || echo '[ -f "$HOME/.proxy_env" ] && . "$HOME/.proxy_env"' >> ~/.profile

grep -qxF '[ -f "$HOME/.proxy_env" ] && . "$HOME/.proxy_env"' ~/.bashrc \
  || echo '[ -f "$HOME/.proxy_env" ] && . "$HOME/.proxy_env"' >> ~/.bashrc
```

立即加载并检查：

```bash
source ~/.proxy_env
env | grep -i proxy
```

### 2.2 测试 OpenAI 网络连接

```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  https://api.openai.com/v1/models
```

返回 `401` 表示服务器已经成功访问 OpenAI，只是该测试请求没有携带认证信息。

### 2.3 配置 Windows VS Code Remote-SSH

安装扩展：

- Remote - SSH
- Codex

打开 SSH 配置文件：

```text
Ctrl + Shift + P
→ Remote-SSH: Open SSH Configuration File
```

示例配置：

```sshconfig
Host <REMOTE_HOST>
  HostName <REMOTE_SERVER_IP>
  User <REMOTE_USER>
  Port 22
```

随后连接 `<REMOTE_HOST>`，打开服务器上的项目目录，并在扩展页面将 Codex 安装到远程主机。

### 2.4 VS Code 代理设置

以下配置建议放在 **Remote 设置** 中，而不是 Windows 全局用户设置中：

```json
{
  "http.proxy": "http://<WINDOWS_PROXY_IP>:7897",
  "http.proxySupport": "override"
}
```

删除手动指定 Windows 可执行文件的配置：

```json
"chatgpt.cliExecutable": "..."
```

删除后，Codex 扩展会自动使用当前远程环境中随扩展提供的 Linux 可执行文件。

---

## 3. Dev Container + Codex

### 3.1 推荐架构

```text
Windows VS Code
  └─ Remote-SSH → Ubuntu（<REMOTE_HOST>）
       └─ Dev Containers Attach → dualmap
            ├─ Codex 扩展及其原生进程运行在容器内
            ├─ 项目、数据、日志和模型通过 bind mount 挂载
            ├─ Codex 网络通过 Windows Clash
            ├─ APT 根据服务器网络情况选择直连或代理
            └─ 必要时使用 device code 登录
```

### 3.2 创建容器

推荐使用 Docker 默认 bridge 网络，并通过 `-p` 显式发布所需端口；不建议使用 `--network host`。

将端口绑定到 `127.0.0.1`，可以避免服务直接暴露到服务器的全部网络接口。需要从 Windows 访问时，可使用 VS Code 的 Ports 面板进行转发。

```bash
docker run -dit \
  --name dualmap \
  --gpus all \
  --shm-size=32g \
  --restart unless-stopped \
  --init \
  -w /workspace/projects/DualMap \
  -p 127.0.0.1:8000-8010:8000-8010 \
  -v /home/<REMOTE_USER>/workspace/data/DualMap:/workspace/data \
  -v /home/<REMOTE_USER>/workspace/logs/DualMap:/workspace/logs \
  -v /home/<REMOTE_USER>/workspace/runs/DualMap:/workspace/runs \
  -v /home/<REMOTE_USER>/workspace/projects/DualMap:/workspace/projects/DualMap \
  -v /home/<REMOTE_USER>/workspace/models:/workspace/models \
  pytorch/pytorch:2.7.1-cuda12.8-cudnn9-devel \
  bash
```

进入容器：

```bash
docker exec -it dualmap bash
```

### 3.3 在容器中配置代理

创建代理配置文件：

```bash
cat > /etc/profile.d/codex-proxy.sh <<'EOF'
export HTTP_PROXY="http://<WINDOWS_PROXY_IP>:7897"
export HTTPS_PROXY="http://<WINDOWS_PROXY_IP>:7897"

export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"

export NO_PROXY="localhost,127.0.0.1,::1"
export no_proxy="$NO_PROXY"
EOF
```

让新的 Bash 会话自动加载：

```bash
grep -qxF 'source /etc/profile.d/codex-proxy.sh' ~/.bashrc \
  || echo 'source /etc/profile.d/codex-proxy.sh' >> ~/.bashrc

source /etc/profile.d/codex-proxy.sh
```

检查环境变量：

```bash
env | grep -i proxy
```

### 3.4 关键：让 APT 强制直连

仅当 Ubuntu 服务器能够直接访问软件源时使用：

```bash
cat > /etc/apt/apt.conf.d/99noproxy <<'EOF'
Acquire::http::Proxy "DIRECT";
Acquire::https::Proxy "DIRECT";
EOF
```

该配置只影响 APT，不会取消其他程序使用的代理环境变量。

### 3.5 安装基础工具

```bash
apt update

apt install -y \
  git \
  curl \
  ca-certificates \
  bubblewrap \
  iproute2 \
  vim
```

### 3.6 测试容器网络

```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  https://api.openai.com/v1/models
```

返回 `401` 表示容器已经能够访问 OpenAI。

---

## 4. Windows VS Code 附加到容器

Windows VS Code 需要安装：

- Remote - SSH
- Dev Containers
- Codex

操作步骤：

1. 通过 Remote-SSH 连接 `<REMOTE_HOST>`。
2. 确认 `dualmap` 容器已经启动。
3. 按 `Ctrl + Shift + P`。
4. 执行 `Dev Containers: Attach to Running Container...`。
5. 选择 `dualmap`。
6. 在容器窗口中打开 `/workspace/projects/DualMap`。
7. 在扩展页面确认 Codex 已安装或启用在 `Dev Container: dualmap`。

进入容器窗口后，VS Code 终端、任务、调试程序和远程扩展均在容器内运行。

---

## 5. 为 VS Code 和 Codex 配置容器代理

仅在 `.bashrc` 中配置代理，主要影响终端 Shell。为了确保 VS Code 扩展宿主、Codex、任务和调试程序也获得代理，应配置 Attached Container 的 `remoteEnv`。

在容器窗口中执行：

```text
Ctrl + Shift + P
→ Dev Containers: Open Named Configuration File
→ 选择 dualmap
```

配置示例：

```json
{
  "workspaceFolder": "/workspace/projects/DualMap",
  "remoteEnv": {
    "HTTP_PROXY": "http://<WINDOWS_PROXY_IP>:7897",
    "HTTPS_PROXY": "http://<WINDOWS_PROXY_IP>:7897",
    "http_proxy": "http://<WINDOWS_PROXY_IP>:7897",
    "https_proxy": "http://<WINDOWS_PROXY_IP>:7897",
    "NO_PROXY": "localhost,127.0.0.1,::1",
    "no_proxy": "localhost,127.0.0.1,::1"
  }
}
```

保存后重新附加容器：

```text
Dev Containers: Reopen in Container
```

也可以关闭当前窗口，再次执行：

```text
Dev Containers: Attach to Running Container...
```

最后检查：

```bash
env | grep -i proxy

curl -s -o /dev/null -w "%{http_code}\n" \
  https://api.openai.com/v1/models
```

参考文档：

- [VS Code：Attach to a running container](https://code.visualstudio.com/docs/devcontainers/attach-container)
- [VS Code：Container environment variables](https://code.visualstudio.com/remote/advancedcontainers/environment-variables)

---

## 6. Codex 登录

### 6.1 浏览器登录

一般情况下，可以直接在 Codex 扩展中使用浏览器登录。

### 6.2 localhost 回调失败

在 Remote-SSH + Dev Container 环境中，浏览器登录有时会出现：

```text
localhost 拒绝连接
ERR_CONNECTION_REFUSED
```

此时使用 device code 登录。

#### 方法一：扩展界面

```text
Codex 登录界面
→ More options
→ Use device code
```

#### 方法二：容器终端

Codex CLI 未安装到系统 `PATH` 时，可直接调用扩展自带的可执行文件：

```bash
/root/.vscode-server/extensions/openai.chatgpt-*/bin/linux-x86_64/codex \
  login --device-auth
```

按照终端提示，在 Windows 浏览器中打开验证页面并输入设备码。登录完成后，可执行：

```text
Developer: Reload Window
```

> 扩展升级后目录版本可能变化。如果系统中残留多个 Codex 扩展版本，建议先确认实际可执行文件路径。

---

## 7. 快速检查清单

### Remote-SSH 环境

```bash
env | grep -i proxy
curl -s -o /dev/null -w "%{http_code}\n" https://api.openai.com/v1/models
```

### 容器环境

```bash
docker ps --filter name=dualmap
docker exec -it dualmap bash
env | grep -i proxy
```

### Codex

```bash
ps aux | grep '[c]odex app-server'
```

如果浏览器回调失败，改用 device code 登录。

