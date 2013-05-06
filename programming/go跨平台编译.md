## 1.生成跨平台的compiler（如linux+amd64）
export GOOS=linux
export GOARCH=amd64
./make.bash

## 2.编译（如linux+amd64）
export GOOS=linux
export GOARCH=amd64
go build source.go

##3.查看文件
file .／source

## 如出现错误：bits/predefs.h找不到，设置CGO不支持
export CGO_ENABLED=0


参考
(http://dave.cheney.net/2012/09/08/an-introduction-to-cross-compilation-with-go)[http://dave.cheney.net/2012/09/08/an-introduction-to-cross-compilation-with-go]

(https://github.com/davecheney/golang-crosscompile/blob/master/crosscompile.bash)[https://github.com/davecheney/golang-crosscompile/blob/master/crosscompile.bash]


 
