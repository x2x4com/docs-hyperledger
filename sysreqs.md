# 系统需求

序号 | 内容 | 更新人 | 更新日期 | 版本
---| --- | --- | --- | ---
1 | 文档初始化 | 许向 | 2019-2-26 | 0.1

## Fabric版本
Release-1.4

## OS
Ubuntu 18.04LTS 4Core 8G 200G

## cURL
保证系统有curl
```
curl -V
```

我的版本 7.58.0

## Docker环境

- Docker 17.06.2-ce+
- Docker Compose version 1.14.0

我的环境

```
docker --version
```

Docker version 18.09.0

docker-compose version 1.17.1

## Golang

Golang 要求1.11x以上，并正确设定GOPATH

```
[ ! -d "$HOME/go" ] && mkdir $HOME/go && echo -e "export GOPATH=\$HOME/go\nexport PATH=\$HOME/go/bin:\$PATH" >> $HOME/.bashrc && source $HOME/.bashrc && echo "Done"|| echo "$HOME/go existed"
```

## NodeJS
要注意的是，Fabric的NodeSDJ并不支持9以上的NodeJS版本，可以安装8.15.x


## Python
18.04默认自带了python3，但是没有python2.7，需要额外安装

```
sudo apt-get install python
```

## Install Samples, Binaries and Docker Images

官方提供了一个在线的脚本方便下载镜像，使用方法

```
curl -sSL http://bit.ly/2ysbOFE | bash -s -- <fabric> <fabric-ca> <thirdparty>
```

下载指定版本

```
curl -sSL http://bit.ly/2ysbOFE | bash -s -- 1.4.0
```


下载后会有这些二进制文件

- configtxgen
- configtxlator
- cryptogen
- discover
- idemixgen
- orderer
- peer
- fabric-ca-client

请把这些文件的路径加入PATH

```
export PATH=<path to download location>/bin:$PATH
```

### 剖析工具脚本

下载这个文件

```
wget "http://bit.ly/2ysbOFE" fabric-bootstrap.sh
```

核心5个部分，分别是

1. 下载Fabric镜像
2. 下载第三方组件镜像
3. 下载CA镜像
4. 下载二进制工具文件
5. 下载Samples

通过3个开关参数可以控制

- `-d` 不下载docker镜像
- `-s` 不下载fabric-samples
- `-b` 不下载二进制文件

### 环境准备工作

在$HOME目录下准备环境

```
[ ! -d $HOME/fabric ] && mkdir $HOME/fabric
```

进入目录，执行一键脚本

```
cd $HOME/fabric && curl -sSL http://bit.ly/2ysbOFE | bash -- 1.4.0
```
