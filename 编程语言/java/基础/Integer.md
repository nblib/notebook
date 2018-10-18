Integer
===
### 在值相同的情况下
* int == int
* int == Integer
* (new Integer) != 任何Integer
* (Integer = Integer.valueof()) != 任何Integer
##### 特殊情况
###### 值小于128
* (Integer = int) == (Integer = int)

###### 值大于128
* (Integer = int) != (Integer = int)
* 


```
        Integer i1 = 59;
        int i2 = 59;
        Integer i3 = Integer.valueOf(59);
        Integer i4 = new Integer(59);
        Integer i7 = new Integer(59);
        int i5 = new Integer(59);
        int i6 = Integer.valueOf(59);
        System.out.println(i1 == i2);
        System.out.println(i1 == i3);
        System.out.println(i1 == i4);
        System.out.println("=============");
        System.out.println(i2 == i3);
        System.out.println(i2 == i4);
        System.out.println(i3 == i4);
        System.out.println("=============");

        System.out.println(i5 == i1);
        System.out.println(i5 == i2);
        System.out.println(i5 == i3);
        System.out.println(i5 == i4);
        System.out.println("=============");

        System.out.println(i6 == i1);
        System.out.println(i6 == i2);
        System.out.println(i6 == i3);
        System.out.println(i6 == i4);
        System.out.println(i6 == i5);
        System.out.println("==============");
        System.out.println(i4 == i1);
        System.out.println(i4 == i2);
        System.out.println(i4 == i3);
        System.out.println(i4 == i5);
        System.out.println(i4 == i6);
        System.out.println(i4 == i7);

        Integer i8 = 277;
        Integer i9 = 277;
        Integer i10 = i8;
        System.out.println("====");
        System.out.println(i8 == i9);
        System.out.println(i8 == i10);
        System.out.println("===");
        int i11 = 277;
        Integer i12 = i11;
        Integer i13 = i11;
        System.out.println(i12 == i13);
```
```
true
true
false
=============
true
true
false
=============
true
true
true
true
=============
true
true
true
true
true
==============
false
true
false
true
true
false
====
false
true
===
false
```