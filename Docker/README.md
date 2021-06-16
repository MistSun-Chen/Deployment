Docker文档：https://docs.docker.com/

# Docker安装

## 卸载旧版本

```bash
 sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 使用存储库安装

### 设置存储库

```bash
 sudo apt-get update
 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

### 安装Docker引擎



#### 安装特定版本DockerEngine

* repo中列出可用版本

```bash
apt-cache madison docker-ce
```

* 选择特定版本安装

```bash
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io

例如：
sudo apt-get install docker-ce=5:18.09.7~3-0~ubuntu-xenial docker-ce-cli=5:18.09.7~3-0~ubuntu-xenial containerd.io
```



* 验证安装是否成功

```bash
sudo docker run hello-world
```

#### 安装最新版本

```bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 从包安装

1. 去[`https://download.docker.com/linux/ubuntu/dists/`](https://download.docker.com/linux/ubuntu/dists/)选择你的Ubuntu版本，然后浏览`pool/stable/`，选择`amd64`， `armhf`或`arm64`，并下载`.deb`文件要安装多克尔引擎版本。

   > **注意**：要安装**nightly**或**test**（预发布）包，`stable`请将上述 URL 中的单词更改为`nightly`或`test`。

2. 安装 Docker Engine，将下面的路径更改为您下载 Docker 包的路径。

   ```bash
   $ sudo dpkg -i /path/to/package.deb
   ```

Docker 守护进程会自动启动。

## 利用脚本安装

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
 DRY_RUN=1 sh ./get-docker.sh
```

# Docker Compose安装

## 安装

1. 运行此命令以下载 Docker Compose 的当前稳定版本：

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> 要安装不同版本的 Compose，请替换`1.29.2` 为您要使用的 Compose 版本。

2. 对二进制文件应用可执行权限：

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

3. 测试安装

```bash
$ docker-compose --version
docker-compose version 1.29.2, build 1110ad01
```

## 升级

如果您从 Compose 1.2 或更早版本升级，请在升级 Compose 后移除或迁移现有容器。这是因为，从 1.3 版开始，Compose 使用 Docker 标签来跟踪容器，并且需要重新创建容器以添加标签。

如果 Compose 检测到创建时没有标签的容器，它会拒绝运行，这样您就不会得到两组。如果您想继续使用现有容器（例如，因为它们有您想要保留的数据卷），您可以使用 Compose 1.5.x 使用以下命令迁移它们：

```bash
docker-compose migrate-to-labels
```

或者，如果您不担心保留它们，则可以删除它们。Compose 只是创建新的。

```bash
docker container rm -f -v myapp_web_1 myapp_db_1 ...
```

## 卸载

如果您使用`curl`以下方式安装，则卸载 Docker Compose ：

```bash
sudo rm /usr/local/bin/docker-compose
```

如果您使用`pip`以下方式安装，则卸载 Docker Compose ：

```bash
pip uninstall docker-compose
```

