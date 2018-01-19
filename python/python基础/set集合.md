set集合
===
1. 集合（set）是一个无序不重复元素的序列。
2. 基本功能是进行成员关系测试和删除重复元素。
3. 可以使用大括号({})或者 set()函数创建集合.
> 注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典

```
student = ({'Tom', 'Jim', 'Mary', 'Tom', 'Jack', 'Rose'})

print(student)  # 输出集合，重复的元素被自动去掉

# 成员测试if('Rose' in student) :
print('Rose 在集合中')else :
print('Rose 不在集合中')
# set可以进行集合运算
a = set('abracadabra')
b = set('alacazam')

print(a)

print(a - b)    # a和b的差集

print(a | b)    # a和b的并集

print(a & b)    # a和b的交集

print(a ^ b)    # a和b中不同时存在的元素
```
