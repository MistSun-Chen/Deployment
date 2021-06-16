Wiki：https://github.com/chrislusf/seaweedfs/wiki

github：https://github.com/chrislusf/seaweedfs

使用方法

## 设置主服务器和卷服务器

### Set up Weed Master

```bash
./weed master -h # to check available options
```

如果不需要复制，这就足够了。“mdir”选项用于配置保存生成的序列文件 id 的文件夹。

```bash
./weed master -mdir="."
./weed master -mdir="." -ip=xxx.xxx.xxx.xxx # usually set the ip instead the default "localhost"
```

### Set up Weed Volume Server

```
./weed volume -h # to check available options
```

通常卷服务器分布在不同的计算机上。它们可以有不同的磁盘空间，甚至不同的操作系统。

通常您需要指定可用磁盘空间、Weed Master 位置和存储文件夹。

```bash
./weed volume -max=100 -mserver="localhost:9333" -dir="./data"
```

## 测试SeaweedFS

主服务器和卷服务器启动后，现在怎么办？让我们将大量文件注入系统！

```bash
./weed upload -dir="/some/big/folder"
```

此命令将递归上传所有文件。或者您可以指定要包含的文件。

```bash
./weed upload -dir="/some/big/folder" -include=*.txt
```

然后，您可以简单地检查“du -m -s /some/big/folder”以查看操作系统的实际磁盘使用情况，并将其与“/data”下的文件大小进行比较。通常，如果您上传大量文本文件，由于文本文件会自动压缩，因此消耗的磁盘大小会小得多。

现在，您可以使用您的工具尽可能地打击 SeaweedFS。