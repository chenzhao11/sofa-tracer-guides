# 使用 SOFATracer 远程汇报数据到 SkyWalking

本示例演示如何在集成了 SOFATracer 的应用，通过配置 SOFATracer 将链路数据远程汇报到Skywalking。

下面的示例中将分别演示在 SOFABoot/SpringBoot 工程中 以及 非 SOFABoot/SpringBoot 工程中如何使用。

## 环境准备

要使用 SOFABoot，需要先准备好基础环境，SOFABoot 依赖以下环境：
- JDK7 或 JDK8
- 需要采用 Apache Maven 3.2.5 或者以上的版本来编译

## 引入 SOFATracer

在创建好一个 Spring Boot 的工程之后，接下来就需要引入 SOFABoot 的依赖，首先，需要将上文中生成的 Spring Boot 工程的 `zip` 包解压后，修改 Maven 项目的配置文件 `pom.xml`，将

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>${spring.boot.version}</version>
    <relativePath/>
</parent>
```

替换为：

```xml
<parent>
    <groupId>com.alipay.sofa</groupId>
    <artifactId>sofaboot-dependencies</artifactId>
    <version>${sofa.boot.version}</version>
</parent>
```

这里的 ${sofa.boot.version} 指定具体的 SOFABoot 版本，参考[发布历史](https://github.com/alipay/sofa-build/releases)。

## 添加 SOFATracer中相关依赖

工程中添加 SOFATracer 依赖：

```xml
 		<dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>tracer-sofa-boot-starter</artifactId>
            <version>3.1.1</version>
            <scope>system</scope>
            <systemPath>${pom.basedir}/src/main/resources/lib/tracer-sofa-boot-starter-3.1.1.jar
            </systemPath>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>tracer-core</artifactId>
            <version>3.1.1</version>
            <scope>system</scope>
            <systemPath>${pom.basedir}/src/main/resources/lib/tracer-core-3.1.1.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-tracer-skywalking-plugin</artifactId>
            <version>${sofa.tracer.version}</version>
            <scope>system</scope>
            <systemPath>
                ${pom.basedir}/src/main/resources/lib/sofa-tracer-skywalking-plugin-3.1.1.jar
            </systemPath>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-tracer-datasource-plugin</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-tracer-flexible-plugin</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-tracer-springmvc-plugin</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-tracer-resttmplate-plugin</artifactId>
        </dependency>
```

因为现在的Jar包在本地所以需要设置<includeSystemScope>为true

```xml
        <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
             <configuration>
                 <includeSystemScope>true</includeSystemScope>
             </configuration>
         </plugin>
```

## 配置文件

最后，在工程的 `application.properties` 文件下添加一个 SOFATracer 要使用的参数，包括`spring.application.name` 用于标示当前应用的名称；`logging.path` 用于指定日志的输出目录。

```properties
# Application Name
spring.application.name=SOFATracerSkyWalking
# logging path
logging.path=./logs
server.port=8082

com.alipay.sofa.tracer.skywalking.enabled=true
com.alipay.sofa.tracer.skywalking.base-url=http://127.0.0.1:12800
com.alipay.sofa.tracer.skywalking.max-buffer-size=10000
```

## 配置其他依赖

使用SkyWalking上传时需要添加如下依赖：

```xml
   		<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.51</version>
        </dependency>
        <!-- opentracing dependency -->
        <dependency>
            <groupId>io.opentracing</groupId>
            <artifactId>opentracing-api</artifactId>
            <version>${opentracing.version}</version>
        </dependency>
        <dependency>
            <groupId>io.opentracing</groupId>
            <artifactId>opentracing-noop</artifactId>
            <version>${opentracing.version}</version>
        </dependency>
        <dependency>
            <groupId>io.opentracing</groupId>
            <artifactId>opentracing-mock</artifactId>
            <version>${opentracing.version}</version>
        </dependency>
        <dependency>
            <groupId>io.opentracing</groupId>
            <artifactId>opentracing-util</artifactId>
            <version>${opentracing.version}</version>
        </dependency>
```

## 启动Skywalking服务端

Skywalking Server可以根据官方的[the user guide for OAP](https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/docker.md) 和 [the user guide for UI](https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/ui-setup.md#start-with-docker-image) 来搭建

## 运行

可以将工程导入到 IDE 中运行生成的工程里面中的 `main` 方法启动应用，也可以直接在该工程的根目录下运行 `mvn spring-boot:run`启动项目。项目启动之后

可以通过在浏览器中输入 http://localhost:8082/helloSkyWalking 来访问 REST 服务，结果类似如下：

```json
{
"success": true,
"id": 1,
"content": "Hello, SOFATracer SkyWalking Remote Report!"
}
```

##  查看链路数据

打开浏览器中访问Skywalking UI界面就可以看到链路信息


## Spring 工程运行

对于一般的 Spring 工程，我们通常使用 tomcat/jetty 作为 servlet 容器来启动应用。具体工程参考 [在 Spring 工程中上传数据到SkyWalking](https://github.com/chenzhao11/tracer-skywalking-plugin-demo)

