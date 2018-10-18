# HUE新建workerflow提交spark项目

## 环境
* CDH6.0
* HUE
* OOZIE

## 开始
### 准备项目
首先准备好需要提交的项目,比如新建一个demo项目: `wordcount`
1. pom.xml
    ```
    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>2.11.12</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.2.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                    <execution>
                        <id>scala-compile-first</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <includes>
                                <include>**/*.scala</include>
                            </includes>
                        </configuration>
                    </execution>
                    <execution>
                        <id>scala-test-compile</id>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ```
2. main方法
    ```
    def main(args: Array[String]): Unit = {
        // create Spark context with Spark configuration
        val sc = new SparkContext(new SparkConf().setAppName("Spark Count"))
    
        // get threshold
        val threshold = args(1).toInt
    
        // read in text file and split each document into words
        val tokenized = sc.textFile(args(0)).flatMap(_.split(" "))
    
        // count the occurrence of each word
        val wordCounts = tokenized.map((_, 1)).reduceByKey(_ + _)
    
        // filter out words with fewer than threshold occurrences
        val filtered = wordCounts.filter(_._2 >= threshold)
    
        // count characters
        val charCounts = filtered.flatMap(_._1.toCharArray).map((_, 1)).reduceByKey(_ + _)
    
        System.out.println(charCounts.collect().mkString(", "))
    }
    ```
    * 通过参数输入要解析的文件,统计文件中字母出现的数量,通过第二个参数控制显示数量超过多少的字母才打印出来.
    
    以上编辑好后,通过maven打包成jar包.
3. 将打包后的jar包上传到hdfs中,比如: `/user/hewe/wordcount-1.0-SNAPSHOT.jar`
### 准备数据
新建一个文件比如`input.txt`:
```
nihao data
name cahge
cache test
tesat hewe
apple
banana counter
counter one two three
three one
five seven eight
twenty one three five counter six
one siz helga
apple banana fiver
```
保存然后上传到hdfs中,比如上传到:`/user/hewe/input.txt`.

### 新建workflow
1. 打开hue管理界面
1. 新建workflow
2. 将`spark program`拖到`Drop your action here`
3. 弹出对话框: Files指向jar包:`/user/hewe/wordcount-1.0-SNAPSHOT.jar`,**注意:通过这个选项选择的路径,可能路径开头没有`/`,导致使用的是相对路径,导致文件找不到**
4. Jar/py name 输入jar包的名称,如上为`wordcount-1.0-SNAPSHOT.jar`
5. 输入完成后,点击Add/添加
6. 添加后会看到填写其他信息.
7. 填写Main class为main方法所在的包名+类名,比如: `demo.hewe.wordcount.App`
8. 由于我们上面的程序需要额外的参数,还需要点击`ARGUMENTS`添加新参数,比如: `hdfs://namenode:8020/user/hewe/input.txt`,以及控制显示数量阈值的`2`,共两个参数.
9. Option list中可以添加需要的额外参数,比如:`--executor-memory 1G`
10. 以上填写好后,点击保存,运行.

### 遇到问题
1. 提示文件不存在和权限问题
    > 出现这个问题,很有可能因为上面选择Files时开头丢失`/`,导致使用的是相对路径.
2. 提示数组越界
    > 配置参数时少了参数,正常情况下,需要两个参数.