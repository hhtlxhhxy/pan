# <img src="https://github.com/hhtlxhhxy/pan/blob/master/img/pan.jpg" alt="image-20200803155136931" style="zoom:50%;" />

-----
## Background
-----
Pan is a high performance and stable production side agent of messager-oriented middleware written in pure Go language. It supports mainstream message queues in the market, such as Kafka, RabbitMQ, RocketMQ, NSQ, etc. Moreover, it is easy to be extended and can meet different business requirements in the production environment.


## Document
-----
[Document](https://tal-tech.github.io/pan-doc/)

## Framework
------
The framework of Pan is shown as below.

<img src="https://github.com/hhtlxhhxy/pan/blob/master/img/fram1.jpg" alt="image-20200803155136931" style="zoom:50%;" />

## Quickstart

### Produce messages to kafka by Pan.
-----

#### 1. Start zookeeper
```shell
./bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties
```
#### 2. Start kafka
```shell
./bin/kafka-server-start /usr/local/etc/kafka/server.properties
```
#### 3. Create topic
```shell
./bin/kafka-topics  --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```
#### 4. Run
```shell
rigger new proxy kafkaproxy
cd $GOPATH/src/kafkaproxy
rigger build
rigger start
```

至此，我们基于pan生成的kafkaproxy就运行起来了，下面介绍如何在业务代码中发送一条message到proxy，然后由proxy转发给kafka

#### 5. Send Message
我们写了一个最简单的发送消息的示例，只有一个main函数，如下
```go
package main
 
import (
    "fmt"
    "time"
 
    "github.com/tal-tech/xtools/kafkautil"

    "github.com/spf13/cast"
)
 
func main() {
    t := time.Tick(5 * time.Second)
    count := 0
    for {
        select {
        case <-t:
            count++
            err := kafkautil.Send2Proxy("test", []byte("kafka "+cast.ToString(count)))
            if err != nil {
                fmt.Println(err)
            }
            continue
        }
    }
}
```
运行这个main，需要单独的配置文件
```shell
;此项配置为业务代码中的配置
[KafkaProxy]
unix=/home/www/pan/pan.sock   //sock in pan
host=localhost:9999  //ip and post pan listen
```

#### warn
replace in go.mod

此处gomod指的是业务项目中的gomod
```shell
replace github.com/henrylee2cn/teleport v5.0.0+incompatible => github.com/hhtlxhhxy/github.com_henrylee2cn_teleport v1.0.0

或

replace github.com/henrylee2cn/teleport v0.0.0 => github.com/hhtlxhhxy/github.com_henrylee2cn_teleport v1.0.0
```
