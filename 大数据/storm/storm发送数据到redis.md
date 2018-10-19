storm发送数据到redis中
===
# 依赖
```
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-redis</artifactId>
    <version>1.1.1</version>
</dependency>
```
# 实现
新建`bolt`继承`AbstractRedisBolt`:
```
package com.usedcar.bolt;

import org.apache.storm.Config;
import org.apache.storm.redis.bolt.AbstractRedisBolt;
import org.apache.storm.redis.common.config.JedisPoolConfig;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Tuple;
import redis.clients.jedis.JedisCommands;

import java.util.Map;

/**
 * @Description: 将数据上传到redis中
 * @Author: hewe
 * @Date:2018/10/19
 * @Project:storm-consumer-kafka
 **/
public class ProdRedisBolt extends AbstractRedisBolt {
    /**
     * 重写构造方法,调用父类的构造
     *
     * @param config
     */
    public ProdRedisBolt(JedisPoolConfig config) {
        super(config);
    }

    //tuple在这里处理
    @Override
    protected void process(Tuple tuple) {
        System.out.println("get a content");
        //获取jedis实例
        JedisCommands jedis = getInstance();
        if (jedis.exists("test")) {
            System.out.println("exists key");
        }
        //用完后,返回到连接池中
        returnInstance(jedis);
    }

    //声明输出字段
    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {

    }

    /**
     * 当需要使用TICK时:
     * 父类默认实现了对TICK的tuple的判断:
     * 当tuple为tick时: 将tuple传入onTickTuple方法
     * 当tuple为自己的数据时: 将tuple传入process 方法.
     * <p>
     * 要实现tick功能,只需要在getComponentConfiguration添加配置.然后在onTickTuple中实现逻辑.
     *
     * @param tuple
     */
    @Override
    protected void onTickTuple(Tuple tuple) {
        System.out.println("this is tick");
    }

    @Override
    public Map<String, Object> getComponentConfiguration() {
        Map<String, Object> configuration = super.getComponentConfiguration();
        configuration.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, 60);
        return configuration;
    }
}
```