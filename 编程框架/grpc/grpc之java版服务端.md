# java版服务端
## 环境
* IDEA
* maven
## 过程
1. 新建一个`maven`项目
1. 添加依赖
    ``` 
    <!-- grpc == START -->
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty</artifactId>
        <version>1.9.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>1.9.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>1.9.0</version>
    </dependency>
    <!--grpc == END -->
    ```
1. 添加插件,用于生成java版的类和方法
    ``` 
     <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.0</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.9.0:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```
1. 在`src`下新建文件夹`proto`,用于存放`.proto`文件,新建一个`hello.proto`文件
    ``` 
    syntax = "proto3"; // 使用语言版本,这里指定使用protobuf3
    
    option java_multiple_files = false; //是否将定义的message,enum等生成一个独立的外部类,如果true,则每个message,enum生成一个class文件,否则,message,enum会统一作为内部类包含在HelloProto类中
    option java_package = "demo.hewe.io.hello"; //定义生成的java类所在的包
    option java_outer_classname = "HelloProto"; //定义这个.proto文件生成的类的名称
    
    package hello; //定义这个.proto文件的作用域,类似java的包的作用,在别的.proto文件引用这个文件中的message时,通过包名.message引用.
    
    //客户端发送参数到服务端,服务端,返回一个message
    // HelloRequest,定义一个message,作为客户端要传递的参数
    message HelloRequest {
        string name = 1; //每一个消息字段都有一个标识编码,比如1,2,3,4,等,可以不按顺序,比如1,5,2,4
        int32 age = 2;
        bool isAdult = 3;
    }
    
    //HelloReply,服务端返回的message
    message HelloReply {
        string receiveTime = 1;
        string info = 2;
    }
    //定义服务类似接口,定义客户端和服务端需要调用/实现的方法
    service HelloSerivce {
        //方法可以传递多个参数,和返回多个参数
        rpc tickInfo (HelloRequest) returns (HelloReply);
    }
    ```
    * `HelloService`为定义的服务接口,里面定义方法.表示服务端需要实现的方法,同时供客户端条用的方法
    * `HelloRequest`和`HelloReply`用于描述方法需要的参数和返回值.
1. 根据`proto`文件生成java版的类和方法.有两种方法:
    * 运行maven的compile命令,点击IDEA的`Maven Projects`的`Lifecycle`的`compile`
    * 分别点击IDEA的`Maven Projects`的`Plugins`->`protobuf`->`protobuf:compile-custom`和`protobuf:compile`
    
    最后在`target`文件夹中`classes`文件夹下或`generated-sources`文件夹下会看到生成的文件
1. 实现生成的类`HelloSerivceGrpc.HelloSerivceImplBase `,并重写定义的方法.具体查看项目`demo-grpc-server`中的`src/main/java/demo.hewe/Services/HelloService`
1. 在主类中注册实现的类,具体看`src/main/java/demo.hewe/App`
1. 最后运行服务端