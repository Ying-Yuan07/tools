# Termux

Termux是 Android 系统的开源终端模拟器。该软件提供了 Linux 环境，即使设备不具备 root 权限也可使用。通过自带的包管理器（Pacman、 APT），Termux 可以安装许多现代化的开发和系统维护工具，例如 zsh、Python、Ruby、NodeJS、MySQL 等软件

## 1. download & install 

1. 从github【1】下载安装包,通过apk安装
2. 通过F-Droid下载【2】

​		PS:**F-Droid**是一个[Android](https://zh.wikipedia.org/wiki/Android)应用程序的软件资源库（或[应用商店](https://zh.wikipedia.org/wiki/应用商店)）；其功能类似于[Google Play](https://zh.wikipedia.org/wiki/Google_Play)商店，但只包含[自由及开放源代码软件](https://zh.wikipedia.org/wiki/自由及开放源代码软件)。

## 2. Xshell远程手机端Termux

ref from ChatGPT 3.5

打开Termux app

### **2.1 在 Termux 上设置 SSH 服务器**

1. **安装 OpenSSH：** 打开 Termux 并输入以下命令安装 OpenSSH：

   ```shell
   pkg update && pkg upgrade
   pkg install openssh
   ```

2. **设置密码：** 设置一个密码以便能够通过 SSH 访问：

   ```shell
   passwd
   ```

3. **启动 SSH 服务器：** 启动 SSH 服务器以允许远程访问：

   ```shell
   sshd
   ```

4. **查看设备 IP 地址：** 你需要知道你的手机的 IP 地址才能从 Xshell 连接到 Termux。可以使用以下命令查看 IP 地址：

   确保你的手机和运行 Xshell 的电脑在同一个 Wi-Fi 网络中，为了安全性，你可以考虑设置密钥认证，而不是密码认证。本例是采用密码认证

   ```shell
   # 安装ip指令
   pkg install iproute2
   ```

   在某些设备上，SELinux 设置可能会阻止某些操作。可以暂时将 SELinux 设置为宽容模式（需要 Root 权限）。在cmd 或者powershell 窗口通过`adb shell`访问手机

   ```shell
   # 避免SELinux阻止ip 指令
   su
   setenforce 0
   ```

   切换到Termux， 查看手机ip

   ```shell
   # 查看设备 IP 地址
   ip addr show
   ```

   记下 wlan0 接口下的 IP 地址（通常是以 `192.168.x.x` 格式）。



5. **检查端口**：和服务器不同，移动设备默认开启8022端口

```shell
ss -tuln | grep 8022
```



6. **查看用户名**: 默认用户名是 `u0_a` 后面跟着一串数字

```shell
whoami
```

### **2.2 在 Xshell 上设置连接**

**打开 Xshell：** 启动 Xshell 应用程序。

**创建新会话：**

- 点击“文件”菜单并选择“新建”。
- 在“新建会话属性”窗口中，填写以下信息：
  - **名称**：给会话起个名字，比如 “Termux SSH”。
  - **协议**：选择 SSH。
  - **主机**：输入你在2.1中获取的 IP 地址。
  - **端口号**：默认是 8022（如果你没有更改 SSH 服务器端口）。

## refs

[1] https://github.com/termux/termux-app/releases

[2] https://f-droid.org/en/