# 系统需求

## 文档版本
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
