# 1. 创建一个zram并开机自动挂起Server

ref from chatgpt 3.5

1. **创建脚本**：

   创建一个脚本来配置 zram。这个脚本将在每次启动时运行。

   ```shell
   sudo nano /usr/local/bin/setup_zram.sh
   ```

   在文件中添加以下内容：

   ```shell
   #!/bin/bash
   
   # Disable all existing swap
   swapoff -a
   
   # Unload any existing zram module
   rmmod zram
   
   # Load zram module with one device
   modprobe zram num_devices=1
   
   # Set the compression algorithm to lz4
   echo lz4 > /sys/block/zram0/comp_algorithm
   
   # Set the zram device size to 50% of the available physical memory
   echo $(($(getconf _PHYS_PAGES) * 4096 / 2)) > /sys/block/zram0/disksize
   
   # Initialize the swap area on the zram device
   mkswap /dev/zram0
   
   # Enable the zram swap
   swapon /dev/zram0
   ```

   保存并退出 (`Ctrl + X`，然后按 `Y`，然后按 `Enter`）。

2. **赋予脚本执行权限**：

   使脚本可执行：

   ```shell
   sudo chmod +x /usr/local/bin/setup_zram.sh
   ```

3. **创建 systemd 服务**：

   创建一个 systemd 服务单元文件，以便在系统启动时运行脚本。

   ```shell
   sudo nano /etc/systemd/system/setup_zram.service
   ```

   在文件中添加以下内容：

   ```shell
   [Unit]
   Description=Setup ZRAM Swap
   After=multi-user.target
   
   [Service]
   Type=oneshot
   ExecStart=/usr/local/bin/setup_zram.sh
   RemainAfterExit=true
   
   [Install]
   WantedBy=multi-user.target
   ```

   保存并退出 (`Ctrl + X`，然后按 `Y`，然后按 `Enter`）。

4. **启用和启动服务**：

   启用服务以便它在启动时运行：

   ```
   sudo systemctl enable setup_zram.service
   ```

   立即启动服务：

   ```shell
   sudo systemctl start setup_zram.service
   ```

5. **验证配置**：

   通过以下命令验证 zram 是否已正确配置：

   ```shell
   sudo swapon --show
   zramctl
   ```

# 2.手机上创建一个swapfile

```shell
cat /proc/swaps
swapoff /dev/block/zram0
dd if=/dev/zero of=/data/local/tmp/swapfile bs=1M count=3072
chmod 600 /data/local/tmp/swapfile
mkswap /data/local/tmp/swapfile
swapon /data/local/tmp/swapfile
free -h
```





修改swapniness

```shell
#修改
oriole:/ # sysctl vm.swappiness=60
vm.swappiness = 60
#查看修改是否成功
oriole:/ # cat /proc/sys/vm/swappiness
60
```





查看swap in/out

```shell
vmstate 1
```



perfetto

```shell
perfetto -c /data/misc/perfetto-configs/config_jank_tiktok.pbtx --txt -o  /data/misc/perfetto-traces/pixel6-no-swap-titok-30s-0bg.perfetto-trace

perfetto -c /data/misc/perfetto-configs/config-new.pbtx --txt -o  /data/misc/perfetto-traces/pixel6-swapflash2g-titok-10s-4bg.perfetto-trace
```



systrace

```shell
python27 F:\1_tools\platform-tools_r31.0.3-windows\systrace\systrace.py -t 600 memory pagecache memreclaim workq sync mmc disk idle freq irq rro database ss dalvik sched gfx view wm am binder_driver -a com.ss.android.ugc.aweme -o tiktok-600s.html
 
python27 F:\1_tools\platform-tools_r31.0.3-windows\systrace\systrace.py -t 6 memory pagecache memreclaim disk dalvik sched view -a com.ss.android.ugc.aweme -o tiktok-no-swap-6s.html
```

