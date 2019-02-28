Ruby被墙，解决办法：

```
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
```

`gem sources -l`查看镜像，输出：
```
*** CURRENT SOURCES ***
https://ruby.taobao.org
```
