# 02_Docker安装

## CentOS安装Docker

参考官网：https://docs.docker.com/engine/install/centos/

### 卸载旧版本

如果之前安装过Docker，需要先卸载旧版本：

```
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

> 旧版本的Docker引擎包可能叫做：`docker`、`docker-engine`。
>
> 新版本的Docker引擎包叫做：`docker-ce`

### 配置yum资源库

安装`yum-config-manager`：

```
# yum-util提供yum-config-manager功能 
sudo yum install -y yum-utils
```

配置docker的资源库地址：

官方地址：（比较慢，不推荐）

```
# 在yum资源库中添加docker资源库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

阿里云镜像地址：

```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> 阿里云官网提供了很多资源镜像，镜像地址：`https://mirrors.aliyun.com`，进入之后可以选择自己需要的资源进行配置

创建缓存（可选）：

```
yum makecache fast
```

### 安装Docker引擎

安装最新版本的Docker引擎、Docker客户端

```
# docker-ce是Docker引擎，docker-ce-cli是客户端
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

此时，默认安装的docker引擎、客户端都是最新版本。

如果要安装指定版本：

```
# 查询版本列表
yum list docker-ce --showduplicates | sort -r

# 指定版本安装17.09.0.ce版
# sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-compose-plugin
sudo yum install docker-ce-17.09.0.ce docker-ce-cli-17.09.0.ce containerd.io docker-compose-plugin
```

### 启动docker引擎

如果没有启动Docker引擎，那么执行 `docker version`查看版本号时，只能看到 `Client: Docker Engine`（Docker引擎客户端）的版本号。

启动Docker引擎：

```
# 新版本的Docker就是一个系统服务，可以直接使用启动系统服务方式启动
systemctl start docker

# 此时查看docker版本，可以看到Server: Docker Engine（Docker引擎）版本号
docker version
```

## 卸载Docker

卸载Docker步骤：

1. 关闭服务

   ```
   systemctl stop docker
   ```

2.  使用`yum`删除docker引擎 

   ```
   sudo yum remove docker-ce docker-ce-cli containerd.io
   ```

3.  删除镜像、容器、卷、自定义配置等文件 

   ```
    sudo rm -rf /var/lib/docker
    sudo rm -rf /var/lib/containerd
   ```

## 运行HelloWorld测试

运行HelloWorld：

```shell
docker run hello-world
```

