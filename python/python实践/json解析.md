# 概述
解析`json`从文件中或字符串中
# code
```
# 从文件中加载json串,加载不到抛IO异常,返回的类型根据json串决定list或map等
data = json.load(open("./procvisorconf.json", "r"))
# 遍历一个集合
for item in data:
    name = item.get("name")
    cmd = item.get("cmd")
    autorestart = item.get("autorestart")
    retrytimes = item.get("retrytimes")

# 如果json中没有赋值,或者赋值为"",都为False
    if not name:
        raise NameError("is null name")
```