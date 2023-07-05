## tips notes

转化静态文件路径

```
def resource_path(relative_path):
    if getattr(sys, 'frozen', False): #是否Bundle Resource
        base_path = sys._MEIPASS
    else:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)
```

在spec文件中加入：

```
datas=[('res', 'res')],
```

在程序中调用静态文件时，引用这个路径

```
self.frm.iconbitmap(resource_path(os.path.join("res", "logo.ico")))
```



curl 请求中需要对JSON中的引号进行转义

```
curl -d "{\"work\":\"12\"}" -H "Content-Type: application/json" -X POST http://localhost:8080
```

codename 查看

```
lsb_release -a
```

更新source.list

```
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

```
https://www.myfreax.com/ubuntu-22-04geng-gai-jing-xiang-ruan-jian-yuan/
```

压缩golang编译文件体积

编译选项

-s: 忽略符号表和调试信息

-w: 忽略DWARFv3调试信息，使用该选项后将无法使用gdb进行调试

使用upx减小体积, 最重要的是压缩率，从1-9

```
upx -9 test.exe
```

使用upx和编译选项组合

```
go build -ldflags="-s -w" -o test.exe main.go && upx -9 test.exe
```

