# 安装kafka过程中遇到的问题

#### 1、zookeeper安装时默认adminServer端口为8080

在zoo.cfg配置文件中添加配置`admin.serverPort=8888`。

在IBMjdk环境下使用默认配置安装kafka会出现如下错误

#### 2、server.properties配置中listeners的IP不能为localhost

listeners配置的是监听地址，需要提供外网服务的话，要设置本地的IP地址，不能设置成localhost。如`listener=PLAINTEXT://84.232.0.1:9092`，PLAINETXT为协议名。

#### 3、IBMjdk环境下安装kafka需要额外配置

~~~shell
<?xml version="1.0" ?>
<verbosegc xmlns="http://www.ibm.com/j9/verbosegc" version="R27_Java727_SR2_20141017_1632_B217728_CMPRSS">
JVMJ9VM007E Command-line option unrecognised: -Xloggc:/opt/kafka_2.10-0.8.2.1/bin/../logs/kafkaServer-gc.log
</verbosegc>
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
~~~

解决方案：

打开bin/kafka-run-class.sh，找到 `KAFKA_GC_LOG_OPTS="-Xloggc:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps "`

把-Xloggc替换成 -Xverbosegclog，如下 `KAFKA_GC_LOG_OPTS="-Xverbosegclog:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps"`