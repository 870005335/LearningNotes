# 04_Docker常用命令

## 启动类命令

启动docker：

```
systemctl start docker
```

停止Docker：

```
systemctl stop docker
```

重启Docker：

```
systemctl restart docker
```

查看状态：

```
systemctl status docker
```

设置开机自启：

```
systemctl enable docker
```

## 帮助类命令

查看Docker版本：

```
docker version
```

查看Docker概要信息：

```
docker info
```

查看Docker总体帮助文档：

```
docker --help
```

查看docker具体命令帮助文档：

```
docker 具体命令 --help
```

## 镜像命令

### 列出本地主机上的镜像

```
docker images
```

参数：

- `-a`：列出所有镜像（含历史镜像）

- `-q`：只显示镜像ID

- `-f`：过滤

### 在远程仓库中搜索镜像

（默认取docker hub中搜索）

```
docker search 镜像名称
```

参数：

- `-f`：过滤

- `--limit 数量`：只展示前几项

### 下载镜像

```
docker pull 镜像名称[:tag]
```

不加 tag 时，默认下载最新的镜像（即tag为`latest`）。

### 导出和导入镜像

`save`：导出镜像的内容流作为一个tar归档文件（对应`save`命令）；

`load`：从tar包中的内容创建一个新的文件系统再导入为镜像（对应`load`命令）；

示例：

```
# 导出
# docker save -o tar文件名 镜像ID
docker save -o aaa.tar abc

# 导入
# cat tar文件 | docker load -i 镜像tar文件名
docker load -i aaa.tar
```



### 查看占据的空间

查看镜像/容器/数据卷所占的空间：

```
docker system df
```

### 删除镜像

```
docker rmi 镜像名称/ID
```

可以使用空格分隔，删除多个镜像：

```
docker rmi 镜像1 镜像2 镜像3
```

删除全部镜像：

```
docker rmi -f ${docker images -qa}
```

### 虚悬镜像

仓库名、标签都是`<none>`的镜像，俗称虚悬镜像（dangling image）。

### 命令自动补全

docker支持命令自动补全功能，当输入镜像名前几位时，可以按`tab`键自动补全镜像名称、tag等。

```
# 如果镜像中有ubuntu，查看输入ub按下tab是否可以补全
docker run ub
```

如果按下`tab`时没有自动补全，可以按以下步骤操作：

1. 检查是否安装了`bash-completion`（命令补全增强包） 

   ```
   # 检查有 /usr/share/bash-completion/bash_completion 这个文件
   ls /usr/share/bash-completion/bash_completion
   ```

2.  如果有`/usr/share/bash-completion`目录，但是没有`/usr/share/bash-completion/bash_completion`文件（centos6为`/etc/bash_completion`文件），则需要安装`bash-completion`

   ```
   yum -y install bash-completion
   ```

3.  检查是否安装了docker的自动补全 

   ```
   # 检查/usr/share/bash-completion/completions文件夹下是否有docker开头的自动补全
   # docker安装完后会在该文件夹下生成自动补全文件docker
   # 如果安装了docker-compose，则该文件夹下还会有 docker-compose文件
   ll /usr/share/bash-completion/completions/docker*
   ```

4. 如果已经安装了docker自动补全，使用`source`命令使其生效

   ```
   source /usr/share/bash-completion/completions/docker
   ```

5.  再次使用`tab`查看是否可以自动补全 

   ```
   # 如果镜像中有ubuntu，查看输入ub按下tab是否可以补全
   docker run ub
   ```

6. 如果有报错，且报错中提示`_get_comp_words_by_ref: command not found`。说明`bash-completion`的配置文件没有生效，需要`source`一下 

   ```
   # 对于centos7，bash-completion安装的是2.x版本，配置文件为/usr/share/bash-completion/bash_completion
   source /usr/share/bash-completion/bash_completion
   
   # 如果是centos6，自动安装的bash-completion最新版为1.x版本，配置文件为/etc/bash_completion
   # bash /etc/bash_completion
   ```

7.  再次使用`tab`查看是否可以自动补全 

   ```
   # 如果镜像中有ubuntu，查看输入ub按下tab是否可以补全
   docker run ub
   ```