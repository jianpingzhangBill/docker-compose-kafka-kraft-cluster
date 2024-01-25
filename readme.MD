# first, init ssl certificate
```
docker-compose -f init.yaml up
```
> it will autocreate ssl certificate include pem and jks.
> if you want to update your password plz update generate.sh line:5 
   and update docker-compose.yaml line:23

> sasl client password: line 18
# start cluster
```
docker-compose up -d 
```


准备：
docker 版本 22 以上
ubuntu 用 `apt install docker.io docker-compose -y`
centos 用 `yum install docker docker-compose -y`

1. 在使用之前，修改 generate.sh 的密码
2. 修改 docker-compose.yaml 的密码
3. docker-compose -f init.yaml 启动

本集群采用 krafa 不再需要 zookeeper。
NOTE: When running in KRaft mode, KAFKA_CFG_SASL_MECHANISM_INTER_BROKER_PROTOCOL only supports PLAIN mechanism in Kafka version <= 3.4. 

主要程序启动 docker-compose 配置文件。支持 stack。这里没有配置内存、CPU 限制，如果要配置此部分内容，参考文档：https://github.com/bitnami/containers/blob/main/bitnami/kafka/README.md

KAFKA_HEAP_OPTS 添加环境变量，-Xmx1024m -Xms1024m

集群包含 python 版本的连接，其他语音请参考 kafka 对应 sdk 包 。


java 测试代码
```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaSaslSslExample {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("security.protocol", "SASL_SSL");
        props.put("sasl.mechanism", "PLAIN");
        props.put("ssl.truststore.location", "/path/to/truststore.jks");   // 修改成生成的 jks 路径，如果是 docker，需要 mount 到容器里
        props.put("ssl.truststore.password", "truststore_password");
        props.put("sasl.jaas.config", "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"kafka_username\" password=\"kafka_password\";");   //修改认证信息

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        String topic = "my_topic";
        String message = "Hello, Kafka!";
        ProducerRecord<String, String> record = new ProducerRecord<>(topic, message);

        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.println("Message sent successfully. Offset: " + metadata.offset());
            } else {
                System.err.println("Error sending message: " + exception.getMessage());
            }
        });

        producer.close();
    }
}
```
