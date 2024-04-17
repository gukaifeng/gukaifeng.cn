假设我们要操作设备 `/dev/sdX`。



## 1. 删除设备



```shell
echo 1 > /sys/block/sdX/device/delete
```



> 如果我们还有把这个设备加回来的打算，则最好记下这个设备的位置：
>
> ```shell
> ll /sys/block/sdX
> ```
>
> 记下输出路径中中的 `hostX`。
>
> 例如下面这个要记下位置 "host0"：
>
> ```shell
> ll /sys/block/sdc
> lrwxrwxrwx 1 root root 0 Apr 17 16:08 /sys/block/sdc -> ../devices/pci0000:17/0000:17:02.0/0000:18:00.0/host0/target0:2:3/0:2:3:0/block/sdc/
> ```



## 2. 添加设备



添加设备就是让内核重新扫描硬件。我们需要先知道所添加的设备位置（就是删除设备时记录的那个 `hostX`）。

```shell
echo '- - -' > /sys/class/scsi_host/hostX/scan
```

注意替换 `hostX`。



> 注意设备号可能不会很快复用。
>
> 例如你的系统里目前有 `/dev/sda` ~  `/dev/sdf`，你删除了 `/dev/sdc` 又很快加了回来，那这个设备的编号可能是 `/dev/sdg`。