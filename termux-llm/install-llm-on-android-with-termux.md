# Install LLM on android with termux 

- 在手机上安装ollama【1】，Ollama 是一个强大的框架，设计用于在 Docker [容器](https://so.csdn.net/so/search?q=容器&spm=1001.2101.3001.7020)中部署 LLM。
- 从hugging face上下载模型，安装transform and pychar

**注**：这两种方式都需要通过Termux，并且需要在 Termux中获取root权限；另外模型下载需要访问外网，需要翻墙，推荐clash。

## 1. Termux 获得root 权限

用 proot-distro 安装 ubuntu（也可以是 debian）

1. 先执行安装

```shell
pkg install proot-distro
```

2. 再安装 debian （也可以是 ubuntu）

```shell
proot-distro install ubuntu
```

3. 再登录 ubuntu，就在termux中获取了root权限

```shell
proot-distro login ubuntu
```



## 2. 在手机上安装ollama，容器化部署

1. 通过xshell远程Termux， 或者也可以在手机上新建会话窗口（在 termux 左上角往右划，左下角有个新建窗口，如果不行就右键长按 termux 应用新建 session）

2. 在 Termux中获取root权限

3. 安装大语言应用工具：ollama

   ```shell
   curl -L -o install.sh https://ollama.com/install.sh
   bash install.sh
   ```

   安装完成后，你可以通过运行以下命令来验证 Ollama 是否正确安装：

   ```shell
   ollama --version
   ```

4. 启动服务：ollama

   ```shell
   #先执行命令启动服务
   ollama serve
   ```

5. 下载&运行模型

    ollama可以下载的模型库，可以在其官网查看【2】

   ```shell
   ollama pull llama3 #从仓库中下载模型
   ollama run llama3 # 执行已经下载到本地的模型，如果本地没有，默认先下载再执行，因此也可以直接执行这条指令
   ```

   

​		

## 3. 从huggingface中下载模型

todo





## refs

[1] https://aizdzj.com/news.php?id=120

[2] https://ollama.com/library