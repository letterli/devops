##### Ramdisk创建

###### RAM Disk

RAM disk在英文里也被称为RAM drive。它将一部分内存分配出来，格式化成一个文件系统(tmpfs)，然后挂载到硬盘的一个目录下，就能像使用硬盘分区一样创建、删除文件和目录。

###### RAM Disk 特点

RAM的速度非常非常地快，即使是现在最快的固态硬盘（SSD），跟内存的速度比起来根本不值得一提。而现在计算机的性能瓶颈就是硬盘。

RAM disk的优点：

> * 非常快
> * 能够进行无数次读取和写入操作

RAM disk的缺点：

> * 内存是易失性存储器，这意味着当计算机关机或重启时，RAM disk里的内容会全部消失。
> * 内存的价格昂贵，所以RAM disk的容量有限。你得注意不要分配太多的空间给RAM disk。

当系统产生大量临时数据或缓存时，如Nginx FastCGI缓存，RAM disk是绝佳的选择。如果你使用固态硬盘（SSD），某些目录需要经常读写时，你可以将这些目录挂载为RAM disk。这样就减少了对固态硬盘的写入次数，延长使用寿命。

###### 创建RAM Disk

首先创建一个目录，这个目录可以在文件系统的任何位置
```shell
sudo mkdir /ramdisk
```

如果你想让所有用户使用这个RAM disk，那么更改目录的权限。
```shell
sudo chmod 777 /ramdisk
```

指定RAM disk的大小，文件系统和设备名，然后将它挂载到一个目录下。
```shell
# 这条命令指定文件系统为tmpfs，RAM disk大小为1024MB，myramdisk是我给它指定的设备名。
sudo mount -t tmpfs -o size=1024m myramdisk /ramdisk
```

###### 测试RAM disk速度

测试RAM disk的写入速度，我们可以用dd工具。测试结果：1.9G/s
```shell
sudo dd if=/dev/zero of=/ramdisk/zero bs=4k count=300000
```

测试RAM disk读取速度：
```shell
sudo dd if=/ramdisk/zero of=/dev/null bs=4k count=300000
```

###### 开机自动挂载RAM Disk

编辑/etc/fstab文件。
```shell
vim /etc/fstab

myramdisk  /tmp/ramdisk  tmpfs  defaults,size=1G,x-gvfs-show  0  0

# x-gvfs-show选项可以让你在文件管理器中看到你的RAM disk。
```
