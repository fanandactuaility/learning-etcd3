# Go 语言安装

## linux安装

在ubuntu之上可以用 apt-get 命令简单安装 `golang-1.6`：

```bash
sudo apt-get install golang-1.6
```

安装完成之后，将go加入到path中，可以简单建立一个软连接：

```bash
sudo ln -s /usr/lib/go-1.6/bin/go go
```

然后执行 `go version` 查看当前版本：

```bash
$ go version
go version go1.6 linux/amd64
```

