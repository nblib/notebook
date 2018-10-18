java中的引用
===
JDK1.2以前,如果reference类型的数据中存储的数值代表另一块内存的起始地址,就称为一个引用.JDK1.2以后,对引用进行了扩充.
* 强引用(Strong Reference)
* 软引用(Soft Reference)
* 弱引用(Weak Reference)
* 虚引用(Phantom Reference)

---

1. 强引用  
类似"Object obj = new Object()",只要强引用还在,垃圾收集器不会回收.
2. 软引用(提供SoftReference类)  
在发生内存溢出之前,会被列进回收范围中进行二次回收,回收后还没有足够的内存,才会抛出内存溢出异常.
3. 弱引用(WeakReference类)  
只能生存到下次回收.无论是否内存充足都会被回收.
4. 虚引用(PhantomReference)  
一个对象是否有虚引用存在,完全不会对其生存时间造成影响,无法通过虚引用获取对象,唯一目的是在被回收之前收到系统通知.