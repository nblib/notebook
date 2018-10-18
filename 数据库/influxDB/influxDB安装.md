# influxDB

## 介绍(翻译自官网)

InfluxDB是一个时间序列数据库，旨在处理高写入和查询负载。 它是TICK堆栈的组成部分。 InfluxDB旨在用作涉及大量带时间戳数据的任何用例的后备存储，包括DevOps监控，应用程序指标，物联网传感器数据和实时分析。

以下是InfluxDB目前支持的一些功能，使其成为处理时间序列数据的绝佳选择:
* 专为时间序列数据编写的自定义高性能数据存储。 TSM引擎允许高摄取速度和数据压缩
* 完全使用go语言编写,并编译为一个单独的二进制文件.没有额外的依赖
* 简单,高性能的读写`HTTP APIS`
* 通过插件支持其他数据提取协议.比如:Graphite,collectd,和OpenTSDB
* 类SQL查询语言,方便数据的查询聚合
* 将tag索引,以便更快的查询速度
* 数据维护策略有效的移除过期数据
* 连续查询自动计算聚合数据,使定期查询更高效

## 安装
1. 下载: 
    ```
    # 适用 RedHat & centos
    # 下载
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.6.3.x86_64.rpm
    #安装
    sudo yum localinstall influxdb-1.6.3.x86_64.rpm
    ```
2. 启动:
    ```
    # centos6.X
    service influxdb start
    # centos7.X
    systemctl start influxdb
    ```
3. 测试:
    ```
    # 在influx安装的机器上运行
    influx
    # 自动连接到本机的influx数据库
    # 显示所有数据库
    SHOW DATABASES
    ```

安装完成