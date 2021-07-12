## 一、环境说明

| ip地址      | 主机名       | 操作系统版本 | RocketMQ版本 |  JDK版本  | maven版本 | 备注            |
| ----------- | ------------ | :----------: | :----------: | :-------: | :-------: | --------------- |
| 172.16.7.91 | nameserver01 |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Name Server集群 |
| 172.16.7.92 | nameserver03 |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Name Server集群 |
| 172.16.7.93 | master01     |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Broker集群1     |
| 172.16.7.94 | slave01      |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Broker集群1     |
| 172.16.7.95 | master02     |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Broker集群2     |
| 172.16.7.96 | slave02      |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    | Broker集群2     |

## 二、部署概况

![image-20210615154528229](https://i.loli.net/2021/07/12/CdIgf86PTEy9n7A.png)

## 三、创建Maven Project

### 1.新建Maven project

![image-20210701093326426](https://i.loli.net/2021/07/12/YGKkuBgXMCpRlnI.png)

![image-20210701093350829](https://i.loli.net/2021/07/12/cixXobm4sSjZWtR.png)

选择Maven Project

![image-20210617162051454](https://i.loli.net/2021/07/12/wyXP2qKnBI5vtkj.png)

配置目录

![image-20210617162115637](https://i.loli.net/2021/07/12/RcouQKXBNWESfIV.png)

选择原型

![image-20210617162140571](https://i.loli.net/2021/07/12/kUhtdrIOzjFx5u9.png)

自定义group id和artifact id，完成maven project的创建。

![image-20210701094557539](https://i.loli.net/2021/07/12/LT4BGA3FIwv1HjW.png)

### 2.导入依赖库

修改pom.xml，加入如下代码

```bash
    <dependency>
      <groupId>org.apache.rocketmq</groupId>
      <artifactId>rocketmq-client</artifactId>
      <version>4.3.0</version>
    </dependency>
```

![image-20210617162433242](https://i.loli.net/2021/07/12/WDGUNoznXL6QZvR.png)

会发现多了很多依赖包

![image-20210617162511598](https://i.loli.net/2021/07/12/8MLw9EoYztsx4PJ.png)

## 四、生产者测试

### 1.测试前集群查看

启动各节点服务，查看集群状态

![image-20210701104356713](https://i.loli.net/2021/07/12/chZFjpwIVexOL3d.png)

测试前无消息生产和消费

### 2.新建topic

#### 2.1新增主题topic_test_123

![image-20210629102044810](https://i.loli.net/2021/07/12/JndvNzItwXZC9FU.png)

主题配置如下：

![image-20210712095049330](https://i.loli.net/2021/07/12/dqo7KXvE4DjfUcJ.png)

集群名为MyRocketmq，BROKER_NAME两个broker都选择

#### 2.2查看新增的主题

![image-20210712095111515](https://i.loli.net/2021/07/12/VX2NKAcLb9GD1o7.png)

### 4.新建订阅组

#### 4.1新建订阅组group_test_123

![image-20210629102224264](https://i.loli.net/2021/07/12/2VLSW1mpagtHlzT.png)

配置如下：

![image-20210712095159712](https://i.loli.net/2021/07/12/uV9eF3IPypwod5j.png)

#### 4.2查看新建的订阅组

![image-20210712095226432](https://i.loli.net/2021/07/12/3PGfhD5VMbjmTNE.png)

### 5.新建类Producer

![image-20210701100726774](https://i.loli.net/2021/07/12/zd41PsES3JTX8hZ.png)

![image-20210617162606023](https://i.loli.net/2021/07/12/dGs3XfM4atWeZQ6.png)

![image-20210617162827728](https://i.loli.net/2021/07/12/zgPinWO7JsYAruv.png)

新建类Producer

![image-20210701095547609](https://i.loli.net/2021/07/12/62gBqKu4ZSErhtU.png)

生产者消息发送代码：

```bash
package com.my.maven.rocketmq;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class Producer {
    public static void main(String[] args) throws Exception {
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new
            DefaultMQProducer("group_test_123");
        // Specify name server addresses.
        producer.setNamesrvAddr("172.16.7.91:9876;172.16.7.92:9876");
        producer.setRetryTimesWhenSendAsyncFailed(2);
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("topic_test_123" /* Topic */,
                "TagA" /* Tag */,
                ("Message Test" +
                    i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}

```

生产者配置项 retryTimesWhenSendAsyncFailed 表示异步重试的次数，默认为 2 次，加上正常发送的1次，总共有3次发送机会。

发送消息Message Test0--Message Test99，共100条消息。

### 6.运行报错

运行Produce发送消息时报错，如图：

![image-20210617164104002](https://i.loli.net/2021/07/12/6aboThzJp4P2sAG.png)

解决：

![image-20210617170017495](https://i.loli.net/2021/07/12/Fv3Pcieb7TUax5o.png)

由于测试是在本地电脑虚机上进行的，同时开多个虚机和eclipse应用会占用很多内存，解决办法是进入eclipse的安装目录，修改文件eclipse.ini，将参数-Xms和-Xmx改小点即可。

### 7.运行Produce

![image-20210712101045969](https://i.loli.net/2021/07/12/8QZTncvf6X7RMr5.png)

### 8.发送消息状态查看

#### 8.1集群查看

![image-20210712101124853](https://i.loli.net/2021/07/12/9rW7fPCeuYkSObi.png)

可以看到broker-a和broker-b各产生了50条消息

#### 8.2消息查看

![image-20210712101213400](https://i.loli.net/2021/07/12/QSYTqWJlf3oBL4c.png)

消息详情：

![image-20210712101231794](https://i.loli.net/2021/07/12/DqOTgkp9mMfetAV.png)

#### 8.3消费者查看

![image-20210712101311181](https://i.loli.net/2021/07/12/j6iATswzDVdf2cg.png)

此时还未消费

## 五、消费者测试

### 1.新建类Consumer

![image-20210617170849639](https://i.loli.net/2021/07/12/xD8voZbJtyq6s7H.png)

消费代码：

```bash
package com.my.maven.rocketmq;

import java.util.List;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

public class Consumer {

    public static void main(String[] args) throws InterruptedException,
            MQClientException {

        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(
                "group_test_123");
        consumer.setNamesrvAddr("172.16.7.91:9876;172.16.7.92:9876");

        consumer.subscribe("topic_test_123", "TagA || TagB");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            public ConsumeConcurrentlyStatus consumeMessage(
                    List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                System.out.println(Thread.currentThread().getName()
                        + " Receive New Messages: " + msgs);
                MessageExt msg = msgs.get(0);
                if (msg.getTopic().equals("topic_test_123")) {
                    if (msg.getTags() != null && msg.getTags().equals("TagA")) {
                        // 获取消息体
                        String message = new String(msg.getBody());
                        System.out.println("receive TagA message:" + message);
                    } else if (msg.getTags() != null
                            && msg.getTags().equals("TagB")) {
                        // 获取消息体
                        String message = new String(msg.getBody());
                        System.out.println("receive TagB message:" + message);
                    }

                }
                // 成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("Consumer Started.");
    }

}
```

### 2.运行Consumer

![image-20210712101455378](https://i.loli.net/2021/07/12/e5cTYLQlvsx1pni.png)

### 3.消费消息状态查看

#### 3.1消费者查看

![image-20210712101615778](https://i.loli.net/2021/07/12/7z86Yo9wpEHjqri.png)

#### 3.2查看消费详情

![image-20210712101658494](https://i.loli.net/2021/07/12/rXVfztNI9Lvlpo2.png)

#### 3.3集群查看

![image-20210712101541424](https://i.loli.net/2021/07/12/fEPgrLjlWT2soY4.png)

#### 3.4消息详情查看

![image-20210712112455003](https://i.loli.net/2021/07/12/eQczStjdEZrYJ93.png)

发现消息已被消费

### 4消费者console日志

![image-20210712152109660](https://i.loli.net/2021/07/12/oNn3kAzadBWKmyO.png)

一共100条消息被消费





单机版RocketMQ搭建详见：[Centos7.6搭建RocketMQ4.8全纪录](https://blog.51cto.com/u_3241766/2908731)

集群版RocketMQ搭建详见：[RocketMQ4.8集群搭建全纪录](https://blog.51cto.com/u_3241766/2930669)

集群启停详见：[RocketMQ集群启停手册](https://blog.51cto.com/u_3241766/2965870)


