time的使用
====
```python
import time

if __name__ == '__main__':
    """
    可以获取年月日等等
    """
    sf: time.struct_time = time.localtime()
    print(sf)
    print(type(sf))
    print(sf.tm_yday) #一年中的天
    print(sf.tm_year) #年
```
