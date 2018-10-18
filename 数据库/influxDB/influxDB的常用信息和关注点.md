# influxDB的端口和配置
此文档包含infuxDB的常用信息,比如:监控的端口,配置文件位置,常用的配置等,方便以后查找和修改.
## 端口
默认influxDB使用以下端口
* `8086`: 用于客户端和服务端交互的HTTP API
* `8088`: 用于提供备份和恢复的RPC服务


## 配置

* 配置文件通过安装包安装,在linux上默认位置:
    ```
    /etc/influxdb/influxdb.conf
    ```

* 查看默认配置:
    ```
    # 列出当前使用的配置
    influxd config
    ```
 * 使用指定配置文件启动
    ```
    influxd -config /etc/influxdb/influxdb.conf
    ```

## 使用路径
* 默认数据保存路径
    ```
    /var/lib/influxdb/data
    ```
* 默认`write-ahead-log(WAL)保存路径
    ```
    /var/lib/influxdb/wal
    ```
* 默认metadata 保存路径
    ```
    /var/lib/influxdb/meta
    ```
    
## 网络时间协议(NTP)
influxDB使用所在主机的本地时间的UTC时间(比国内晚8个小时)来设置timestamp,多个主机之间使用NTP协议同步时间,如果时间不同步,会导致数据的时间戳不准确.
