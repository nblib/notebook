# 概述

shlex类可以轻松地为类似于Unix shell的简单语法编写词法分析器。这通常用于编写minilanguages（例如，在Python应用程序的运行控制文件中）或用于解析引用的字符串。
# code
``` 
print(shlex.split("ls -a \"./asdf\""))
```
结果:
``` 
['ls', '-a', './asdf']
```
* 双引号被去掉
* 按空字符分开