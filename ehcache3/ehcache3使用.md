# ehcache3使用
## 基本使用
### 环境
* java8
* maven
### xml配置
```
<config
        xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'
        xmlns='http://www.ehcache.org/v3'
        xsi:schemaLocation="http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core.xsd">
    <!--缓存到磁盘的位置-->
    <persistence directory="d:\\mycache"/>

<!--缓存模板,每一个缓存可以基于这个模板添加修改,或直接使用-->
    <cache-template name="myDefaults">
        <!--缓存key的类型,不设置默认为Object-->
        <key-type>java.lang.String</key-type>
        <!--值类型,不设置默认为Object,serializer设置序列化工具类,不设置的话,object无法序列化,这里使用的是java的序列化工具,即需要实现序列化接口,不然为空-->
        <value-type serializer="org.ehcache.impl.serialization.PlainJavaSerializer">java.lang.Object</value-type>
        <!--缓存过期时间,可以是tti(空闲保留时间),ttl(存活时间),None(不过期),默认不过期-->
       <!-- <expiry>
            <tti unit="minutes">20</tti>
        </expiry>-->
        <!--缓存保存类型-->
        <resources>
            <!--在JVM中保存缓存,可以按照实体个数,总大小设置-->
            <heap unit="entries">20000</heap>
            <!--缓存到磁盘的设置,可以是kb,Mb,GB等单位设置大小-->
            <disk unit="GB" persistent="true">3</disk>
        </resources>

    </cache-template>

    <!--一个缓存实例,基于模板-->
    <cache alias="foo" uses-template="myDefaults"></cache>
</config>
```
### java代码
#### 用到的测试类
```
package demo.hewe;

import java.io.Serializable;
import java.util.Date;

/**
 * @year 2018
 * @project demo-ehcache
 * @description:<p></p>
 **/
public class Bar implements Serializable{
    private String name;
    private int age;
    private Date created;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Date getCreated() {
        return created;
    }

    public void setCreated(Date created) {
        this.created = created;
    }
}

```
#### 主方法
```
 public static void main( String[] args )
    {
        URL myUrl = App.class.getClassLoader().getResource( "ehcache.xml");
        Configuration xmlConfig = new XmlConfiguration(myUrl);
        CacheManager myCacheManager = CacheManagerBuilder.newCacheManager(xmlConfig);
        myCacheManager.init();//必须调用这个方法初始化,不然会报错
        Cache<String, Object> foo = myCacheManager.getCache("foo", String.class, Object.class);
        foo.put("ni",new Bar());
        /*foo.put("na","bao");
        foo.put("nq","qao");*/

        Bar ni = (Bar)foo.get("ni");
        Object na = foo.get("na");
        System.out.println(ni);
        System.out.println(na);
    }
```
## 介绍
ehcache3比2变化较大.

每个缓存可以单独设置,也可以基于一个模板.

### 保存缓存方式
* heap : java堆中保存.
* off-heap: 非堆保存
* disk: 磁盘保存

这三种方式可以单独使用,也可以结合使用.结合使用时,占用大小设置关系: heap < off-heap < disk.本地测试使用heap+disk,发现缓存会**同时存在heap和disk中,heap中装不下的保存在disk中.**
#### 大小设置方式
##### heap
* entries: 表示这个缓存中的键值对数量
* kb: 缓存的大小,以kb为单位
* MB: 同于kb,以MB为单位

...
##### disk
* kb: 缓存的大小,以kb为单位
* MB,GB,等: 缓存的大小单位
### 过期设置
* ttl: 缓存存活时间
* tti: 缓存空闲时间
* None : 不过期

过期单位可以是秒,分,小时,等等.

## 遇到问题
### Caused by: org.ehcache.StateTransitionException: Initial table allocation failed.
保存到磁盘的大小设置过小,只用一个缓存的时候,初始化需要的磁盘大小为16k,如果设置的值小于这个会报错.
### org.ehcache.spi.serialization.UnsupportedTypeException: No serializer found for type 'java.lang.Object'
当缓存value使用Object时,ehcache会在内置的序列化类中寻找适用于Object的类.找不到然后报这个错误,即使实现类实现了序列化接口,这样也会报错,
#### 解决方法(这里列出两种)
* 修改value的类型为`java.io.Serializable`
* 手动指定序列化类` <value-type serializer="org.ehcache.impl.serialization.PlainJavaSerializer">java.lang.Object</value-type>`
