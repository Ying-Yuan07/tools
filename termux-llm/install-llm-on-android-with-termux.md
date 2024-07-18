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

1. **通过xshell远程Termux， 或者也可以在手机上新建会话窗口（在 termux 左上角往右划，左下角有个新建窗口，如果不行就右键长按 termux 应用新建 session）**

2. **在 Termux中获取root权限**

3. **安装大语言应用工具：ollama**

   ```shell
   curl -L -o install.sh https://ollama.com/install.sh
   bash install.sh
   ```

   安装完成后，你可以通过运行以下命令来验证 Ollama 是否正确安装：

   ```shell
   ollama --version
   ```

   

4. **启动服务：ollama**

   ```shell
   #先执行命令启动服务
   ollama serve
   ```

5. **下载&运行模型**

   在xshell或者手机上新开启一个 Termux会话窗口，进入root，下载并运行模型

    ollama可以下载的模型库，可以在其官网查看【2】

   ```shell
   ollama pull llama3 #从仓库中下载模型
   ollama run llama3 # 执行已经下载到本地的模型，如果本地没有，默认先下载再执行，因此也可以直接执行这条指令
   ```

   等进度条跑完下载就行了，可以愉快进行测试了

 6. **测试**

    执行`ollama serve`启动ollama，通过下面日志我们可以看到， Ollama 已准备好接收来自 `llama3` 模型的请求，并且可以进行推理任务。可以通过发送 HTTP 请求到 `127.0.0.1:11434` 或 `127.0.0.1:38971` 来与模型进行交互。

    **端口 11434**：

    - 这是 Ollama 服务器的主要监听端口，用于接收管理和控制请求。通常，您会通过这个端口与 Ollama 的管理接口进行交互，比如管理模型、查看服务器状态或配置。

    **端口 38971**：

    - 这是具体模型服务的端口。当您进行推理或运行模型时，Ollama 会在这个端口上启动一个服务，用于接收来自客户端的推理请求，传递输入数据并返回模型的输出结果。

```shell
root@localhost:~# ollama serve
2024/07/05 02:36:47 routes.go:1064: INFO server config env="map[CUDA_VISIBLE_DEVICES: GPU_DEVICE_ORDINAL: HIP_VISIBLE_DEVICES: HSA_OVERRIDE_GFX_VERSION: OLLAMA_DEBUG:false OLLAMA_FLASH_ATTENTION:false OLLAMA_HOST:http://127.0.0.1:11434 OLLAMA_INTEL_GPU:false OLLAMA_KEEP_ALIVE: OLLAMA_LLM_LIBRARY: OLLAMA_MAX_LOADED_MODELS:1 OLLAMA_MAX_QUEUE:512 OLLAMA_MAX_VRAM:0 OLLAMA_MODELS:/root/.ollama/models OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:1 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://*] OLLAMA_RUNNERS_DIR: OLLAMA_SCHED_SPREAD:false OLLAMA_TMPDIR: ROCR_VISIBLE_DEVICES:]"
time=2024-07-05T02:36:47.302Z level=INFO source=images.go:730 msg="total blobs: 5"
time=2024-07-05T02:36:47.309Z level=INFO source=images.go:737 msg="total unused blobs removed: 0"
time=2024-07-05T02:36:47.317Z level=INFO source=routes.go:1111 msg="Listening on 127.0.0.1:11434 (version 0.1.48)"
time=2024-07-05T02:36:47.322Z level=INFO source=payload.go:30 msg="extracting embedded files" dir=/tmp/ollama3197932021/runners
time=2024-07-05T02:36:51.164Z level=INFO source=payload.go:44 msg="Dynamic LLM libraries [cpu cuda_v11]"
time=2024-07-05T02:36:51.217Z level=INFO source=types.go:98 msg="inference compute" id=0 library=cpu compute="" driver=0.0 name="" total="7.4 GiB" available="5.0 GiB"
[GIN] 2024/07/05 - 02:36:55 | 200 |     196.452µs |       127.0.0.1 | GET      "/api/version"
[GIN] 2024/07/05 - 02:37:03 | 200 |      66.121µs |       127.0.0.1 | HEAD     "/"
[GIN] 2024/07/05 - 02:37:04 | 200 |  275.807699ms |       127.0.0.1 | POST     "/api/show"
time=2024-07-05T02:37:04.449Z level=INFO source=memory.go:309 msg="offload to cpu" layers.requested=-1 layers.model=33 layers.offload=0 layers.split="" memory.available="[5.0 GiB]" memory.required.full="5.1 GiB" memory.required.partial="0 B" memory.required.kv="256.0 MiB" memory.required.allocations="[4.7 GiB]" memory.weights.total="3.9 GiB" memory.weights.repeating="3.5 GiB" memory.weights.nonrepeating="411.0 MiB" memory.graph.full="164.0 MiB" memory.graph.partial="677.5 MiB"
time=2024-07-05T02:37:04.476Z level=INFO source=server.go:368 msg="starting llama server" cmd="/tmp/ollama3197932021/runners/cpu/ollama_llama_server --model /root/.ollama/models/blobs/sha256-6a0746a1ec1aef3e7ec53868f220ff6e389f6f8ef87a01d77c96807de94ca2aa --ctx-size 2048 --batch-size 512 --embedding --log-disable --no-mmap --parallel 1 --port 38971"
time=2024-07-05T02:37:04.498Z level=INFO source=sched.go:382 msg="loaded runners" count=1
time=2024-07-05T02:37:04.499Z level=INFO source=server.go:556 msg="waiting for llama runner to start responding"
time=2024-07-05T02:37:04.501Z level=INFO source=server.go:594 msg="waiting for server to be
```



使用 `curl` 命令来发送一个简单的 HTTP POST 请求到 Ollama 服务器的推理服务端口，会收到一个模型回答的 JSON 响应。

```shell
curl -X POST http://127.0.0.1:11434/api/generate -d '{"model": "llama3", "prompt": "What is your name?"}'
```

输出结果如下

```shell
root@localhost:/# curl -X POST http://127.0.0.1:11434/api/generate -d '{"model": "llama3", "prompt": "What is your name?"}'
{"model":"llama3","created_at":"2024-07-05T02:55:29.685093129Z","response":"I","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:30.230579172Z","response":" don","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:30.630516347Z","response":"'t","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:30.996778473Z","response":" have","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:31.858895539Z","response":" a","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:32.648765697Z","response":" personal","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:33.104574779Z","response":" name","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:33.466287467Z","response":".","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:33.84282185Z","response":" I","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:34.232475496Z","response":" am","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:35.330316439Z","response":" an","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:36.647039299Z","response":" artificial","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:37.030922153Z","response":" intelligence","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:37.471908888Z","response":" language","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:37.854742262Z","response":" model","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:38.266121942Z","response":" designed","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:38.65668021Z","response":" to","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:39.229253249Z","response":" assist","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:39.622835118Z","response":" and","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:40.303427607Z","response":" communicate","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:41.03045019Z","response":" with","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:41.422742467Z","response":" users","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:42.281319209Z","response":" in","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:42.672030106Z","response":" a","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:43.08913529Z","response":" helpful","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:43.507248816Z","response":" and","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:44.404410519Z","response":" informative","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:44.820184648Z","response":" way","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:45.218337033Z","response":".","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:45.635658729Z","response":" You","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:46.060741289Z","response":" can","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:46.47747245Z","response":" think","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:47.322680011Z","response":" of","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:47.73429549Z","response":" me","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:48.250121377Z","response":" as","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:48.661630492Z","response":" a","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:49.081247232Z","response":" convers","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:49.668467569Z","response":"ational","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:50.088155273Z","response":" AI","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:51.110493489Z","response":",","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:52.0245177Z","response":" and","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:52.63114913Z","response":" you","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:53.300953492Z","response":" can","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:53.725626995Z","response":" refer","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:54.15733838Z","response":" to","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:54.588176352Z","response":" me","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:55.024573569Z","response":" as","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:55.471177044Z","response":" \"","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:55.902565065Z","response":"Assistant","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:56.327334719Z","response":"\"","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:56.826634158Z","response":" or","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:57.255836713Z","response":" \"","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:57.817351769Z","response":"AI","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:58.252348148Z","response":"\"","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:58.69276693Z","response":" if","done":false}
{"model":"llama3","created_at":"2024-07-05T02:55:59.612194665Z","response":" you","done":false}
{"model":"llama3","created_at":"2024-07-05T02:56:00.442894006Z","response":" like","done":false}
{"model":"llama3","created_at":"2024-07-05T02:56:00.883376429Z","response":"!","done":false}
{"model":"llama3","created_at":"2024-07-05T02:56:01.329202479Z","response":"","done":true,"done_reason":"stop","context":[128006,882,128007,271,3923,374,701,836,30,128009,128006,78191,128007,271,40,1541,956,617,264,4443,836,13,358,1097,459,21075,11478,4221,1646,6319,311,7945,323,19570,449,3932,304,264,11190,323,39319,1648,13,1472,649,1781,315,757,439,264,7669,1697,15592,11,323,499,649,8464,311,757,439,330,72803,1,477,330,15836,1,422,499,1093,0,128009],"total_duration":37806489805,"load_duration":2107458009,"prompt_eval_count":15,"prompt_eval_duration":4050415000,"eval_count":59,"eval_duration":31641240000}
```



## 3. 从huggingface中下载模型

用xshell远程连接Termux，并进入root， 进入虚拟环境

### 3.1 安装必要的库

安装`python3`，`pip`

```shell
apt update
apt upgrade -y
apt install python3
apt install python3-pip
```

模型运行一般需要`torch`和`transformers`， 测试模型需要数据库`datasets` ， 为了防止系统Python环境被破坏，我们使用虚拟环境

1. **创建虚拟环境**：

```
python3 -m venv myenv
```

2. **激活虚拟环境**：

```
source myenv/bin/activate
```

xshell界面显示如下，即已进入虚拟环境

```shell
(myenv) root@localhost:~#
```

3. **安装库**：

```shell
pip install transformers
pip install torch
pip install datasets
```



### 3.2 从hugging face上下载模型，并传输到手机

在这里我们选择`gemma-1.1-2b-it`【3】,下载模型需要在huggingface中登录账号，有些还需要申请，注册与申请信息都写美国（写中国可能被拒），点击`Files and versions`，如果可以看到文件，则说明可以直接下载或者已经通过了申请，直接在网页上就可以下载该模型，下载该目录下所有的文件，并通过Xftp传输到手机。如果需要申请，申请通过一般需要等待一个小时左右。



## 3.3 运行模型

huggingface网站`gemma-1.1-2b-it`主页`Model card`上给出了模型运行demo，由于我们已经把模型下载到了本地，将demo中模型路径修改为本地路径即可。

**注**：gemma-1.1-2b-it支持用TPU运行【5】，但是需要`PyTorch/XLA`【6】,目前只支持x86，不支持arm【7】，因此在手机上暂时只能运行CPU与GPU推理版本

### 3.3.1在CPU上运行模型

参考官方[demo](https://huggingface.co/google/gemma-1.1-2b-it)

### 3.3.2 在GPU上运行模型【尚未解决】

**from ChatGPT3.5**

[google](https://huggingface.co/google)/[gemma-1.1-2b-it](https://huggingface.co/google/gemma-1.1-2b-it) huggingface给的demo不能直接运行，因为demo使用CUDA运行在GPU上，仅支持英伟达；而采用`AIDA64`查看pixel6的GPU参数，得知 Pixel 6 使用的是基于 ARM 的 GPU（Mali-G78，openGL es3.2），它们不支持 CUDA。因此，需要调整代码以适应手机的 GPU 环境。

**尝试使用 TensorFlow Lite**：

TensorFlow Lite 是为移动和嵌入式设备设计的轻量级框架，可以将模型转换为适合移动设备运行的格式。你可以尝试将 PyTorch 模型转换为 TensorFlow，然后使用 TensorFlow Lite 运行。

以下是将 PyTorch 模型转换为 TensorFlow Lite 的基本步骤：

1. **将 PyTorch 模型转换为 ONNX**：

   ```python
   import torch
   from transformers import AutoModelForCausalLM
   
   model_path = "/data/data/com.termux/files/home/google/gemma-1.1-2b-it"
   model = AutoModelForCausalLM.from_pretrained(model_path)
   
   dummy_input = torch.randint(0, 2, (1, 3))
   torch.onnx.export(model, dummy_input, "model.onnx")
   ```

2. **将 ONNX 模型转换为 TensorFlow**：

   安装必要的包：

   ```
   pip install onnx onnx-tf
   ```

   转换模型：

   ```python
   import onnx
   from onnx_tf.backend import prepare
   
   onnx_model = onnx.load("model.onnx")
   tf_rep = prepare(onnx_model)
   tf_rep.export_graph("model.pb")
   ```

3. **将 TensorFlow 模型转换为 TensorFlow Lite**：

   ```python
   import tensorflow as tf
   
   converter = tf.lite.TFLiteConverter.from_saved_model("model.pb")
   tflite_model = converter.convert()
   
   with open("model.tflite", "wb") as f:
       f.write(tflite_model)
   ```

   然后可以在手机上使用 TensorFlow Lite 运行该模型。



完整的文件路径如下

```shell
/my_project
│
├── convert_model.py      # 用于将 PyTorch 模型转换为 TensorFlow Lite 模型
├── run_tflite_model.py   # 用于在 Android 设备上运行 TensorFlow Lite 模型
├── model.onnx            # 转换过程中的中间文件
├── model.tflite          # 最终的 TensorFlow Lite 模型文件
```



#### 将 PyTorch 模型转换为 TensorFlow Lite 模型【Failed】

**注**：在手机上执行`convert_model.py` 脚本失败，报错：`ModuleNotFoundError: No module named 'tensorflow_addons'`,在手机上安装`tensorflow_addons`失败



以下是用于在计算机上进行模型转换的 `convert_model.py` 脚本：

```python
import torch
from transformers import AutoModelForCausalLM
import onnx
from onnx_tf.backend import prepare
import tensorflow as tf

# 设置模型路径
model_path = "/data/data/com.termux/files/home/google/gemma-1.1-2b-it"
model = AutoModelForCausalLM.from_pretrained(model_path)

# 1. 将 PyTorch 模型转换为 ONNX
dummy_input = torch.randint(0, 2, (1, 128))
torch.onnx.export(model, dummy_input, "model.onnx")

# 2. 将 ONNX 模型转换为 TensorFlow
onnx_model = onnx.load("model.onnx")
tf_rep = prepare(onnx_model)
tf_rep.export_graph("model")

# 3. 将 TensorFlow 模型转换为 TensorFlow Lite
converter = tf.lite.TFLiteConverter.from_saved_model("model")
tflite_model = converter.convert()

# 保存 TensorFlow Lite 模型
with open("model.tflite", "wb") as f:
    f.write(tflite_model)
```

运行这个脚本会生成 `model.onnx` 和 `model.tflite` 文件。

#### 将模型文件复制到 Android 设备

1. **将 `model.tflite` 文件复制到 Android 设备**：

   使用 `adb` 将文件复制到 Android 设备：

   ```python
   adb push model.tflite /data/local/tmp/
   ```

#### 在 Android 设备上运行 TensorFlow Lite 模型

在 Android 设备上，创建一个新的 Python 脚本来加载和运行 TensorFlow Lite 模型：

```python
import numpy as np
import tensorflow as tf

# 加载 TensorFlow Lite 模型
interpreter = tf.lite.Interpreter(model_path="/data/local/tmp/model.tflite")
interpreter.allocate_tensors()

# 获取输入和输出张量的详细信息
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# 准备输入数据
input_data = np.random.randint(0, 2, size=(1, 128)).astype(np.float32)

# 将输入数据写入输入张量
interpreter.set_tensor(input_details[0]['index'], input_data)

# 运行推理
interpreter.invoke()

# 获取输出数据
output_data = interpreter.get_tensor(output_details[0]['index'])
print("Output:", output_data)
```

#### 运行步骤

1. **在计算机上运行 `convert_model.py`**：

   ```
   python convert_model.py
   ```

   这将生成 `model.onnx` 和 `model.tflite` 文件。

2. **将 `model.tflite` 文件复制到 Android 设备**：

   ```
   adb push model.tflite /data/local/tmp/
   ```

3. **在 Android 设备上运行 `run_tflite_model.py`**：

   ```
   python /data/local/tmp/run_tflite_model.py
   ```



安装依赖`onnx onnx-tf tensorflow`,由于`tensorflow`依赖`h5py`,先安装`h5py`

```shell
apt update
apt upgrade
apt install build-essential libhdf5-dev pkg-config
pip install h5py #耗时较久
```

安装完成后，你可以验证 `h5py` 是否成功安装：

```shell
python -c "import h5py; print(h5py.__version__)"
```

这将打印安装的 `h5py` 版本，确认安装成功。

安装依赖`onnx onnx-tf tensorflow`

```shell
pip install onnx onnx-tf tensorflow
```



## 3.4 用公开数据库测试模型

在`gemma-1.1-2b-it`主页`Model card`上看到，其测试benchmark 包括`MMLU`【4】, 和下载模型类似，我们也将`MMLU`数据库下载到本地







## 3.5 查看swap info



```shell
#1 系统IO状态信息
iostat -x 1 
```





## 3.6 限制内存资源





在服务器上【8】



```shell
vim /etc/security/limits.conf
#在尾部插入
@yy              hard    rss             50000000 #限制yy用户rss 为5GB
```



```shell
vim /etc/pam.d/login
#在尾部插入
session    required            /lib/x86_64-linux-gnu/security/pam_limits.so
```

检查设置是否生效

```shell
su yy
ulimit -a
```

```shell
yy@tan08:/lib$ ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 1029883
max locked memory           (kbytes, -l) 32970832
max memory size             (kbytes, -m) 50000000 #设置内存使用上限成功
open files                          (-n) 1024
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 1029883
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```





模型在termux终端中运行，也属于termux对应的用户，因此我们在Termux终端中通过` ulimit  -a`查看该会话的所有限制，也是模型运行的限制，可以看到占用内存大小无限制，**通过Termux 设置失败，因为 Termux 的环境与标准的 Linux 环境有所不同，并且某些系统级操作可能无法在 Termux 中实现**

```shell
 ulimit  -a
```

```shell
#通过ulimit查看
root@localhost:~# ulimit  -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 40
file size                   (blocks, -f) unlimited
pending signals                     (-i) 27544
max locked memory           (kbytes, -l) 64
max memory size             (kbytes, -m) unlimited  
open files                          (-n) 32768
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 27544
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```



cgroup

```shell
#查看支持的cgroup
mount | grep cgroup
cat /proc/cgroups
```

```shell
oriole:/sys/fs/cgroup/uid_10117 # mount | grep cgroup
none on /dev/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
none on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,memory_recursiveprot)
none on /dev/cpuctl type cgroup (rw,nosuid,nodev,noexec,relatime,cpu)
none on /dev/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset,noprefix,release_agent=/sbin/cpuset_release_agent)
```



```shell
130|oriole:/ $ cat /proc/cgroups
#subsys_name    hierarchy       num_cgroups     enabled
cpuset  3       10      1
cpu     2       10      1
cpuacct 0       205     1
blkio   1       2       1
memory  0       205     0 #????
freezer 0       205     1
net_prio        0       205     1
```



根据教程https://cloud.baidu.com/article/3002492

```
mkdir /sys/fs/cgroup/mygroup
```

但是该路径下并没有`memory.limit_in_bytes`，并且`cgroup.controllers`内容为空，

另外，可以通过` cgroup.freeze`,冻结对应用户的所有进程

```shell
oriole:/sys/fs/cgroup/uid_10117 # echo 1 >> cgroup.freeze
```







https://serverfault.com/questions/929080/list-of-controllers-empty-with-cgroup-v2

https://android.googlesource.com/kernel/common/+/429fec28c835/Documentation/admin-guide/cgroup-v2.rst

https://medium.com/@charles.vissol/cgroup-v2-in-details-8c138088f9ba

https://medium.com/@charles.vissol/cgroup-v2-in-details-8c138088f9ba



## refs

[1] https://aizdzj.com/news.php?id=120

[2] https://ollama.com/library

[3] https://huggingface.co/google/gemma-1.1-2b-it/tree/main

[4] https://huggingface.co/datasets/hails/mmlu_no_train

[5] https://huggingface.co/google/gemma-7b/blob/main/examples/example_fsdp.py

[6] https://github.com/pytorch/xla

[7] https://storage.googleapis.com/pytorch-xla-releases

[8] https://blog.csdn.net/ranguangrong/article/details/116156885