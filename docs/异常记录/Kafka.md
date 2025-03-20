## Kafka连接失败

1. **检查 Kafka 服务器是否运行：** 确保Kafka 服务器正在运行，并且网络连接正常。

   - 可以使用**命令行工具**（ `kafka-topics.sh` 或 `kafka-console-consumer.sh` ）来检查 Kafka 服务器的状态。

2. **检查防火墙设置：** 确保防火墙或网络配置没有阻止消费者与 Kafka 服务器之间的通信。

   - 确保端口 9092（默认 Kafka broker 端口）是开放的。

3. **检查Kafka配置：** 确保在消费者代码中提供的 Kafka 服务器地址和端口是正确的。

   - 检查 `bootstrap.servers` 参数是否正确设置为 Kafka 服务器的地址和端口。

   ```
   codeconsumer_config = {
       'bootstrap.servers': 'your_kafka_bootstrap_servers',
       # 其他配置...
   }
   ```

4. **Kafka版本兼容性：** 确保使用的 **Kafka消费者库与 Kafka 服务器版本兼容**。不同版本之间的不匹配可能会导致问题。

5. **网络可达性：** 确保消费者所在的主机可以访问 Kafka 服务器的网络。如果有防火墙、代理或网络隔离，确保它们没有导致连接问题

   - 以开启sasl_ssl访问的Kafka实例为例，执行如下命令：`curl -kv {ip}:{port}`

## offset commit failed on partition the request timed out

根据Kafka 报错 ，通常意味着 Kafka 消费者在尝试提交偏移量时超时了。

可能的原因包括：

- 网络延迟或不稳定：消费者与 Kafka 集群之间的网络连接不稳定或延迟较高。
- Kafka 配置不合理：某些配置参数设置不当，导致消费者无法在规定时间内完成偏移量提交。
- 消费者处理时间过长：消费者处理消息的时间超过了 Kafka 的会话超时时间（session.timeout.ms)），导致 Kafka 认为消费者已死亡并重新分配分区。

## 不消费消息问题（消费者组中多个消费者）

如果发现一个**消费者客户端A**已经启动了，但是就是不消费消息

此时应该检查一下该**消费者组**中是否还有**其它消费者**

消息可能被**消费者组**中其它消费者线程**抢走**（负载均衡机制）

## 新消费组需要注意的问题（latest+新消费组）

背景：kafka偏移量策略设置为latest，场景重现：

1. 先生产消息
2. 然后再启动服务（使用**新消费组**）
3. 没有收到消息

原因

- 因为使用了新消费组，偏移量策略设置为latest，**消费者组**创建之前的消息都不消费(不消费旧消息)

解决：

- 先启动服务（创建新消费组），再生产消息。（注意服务启动顺序）
  - **服务启动一次后**，**新消费组已经被创建**，之后只要服务启动都会自动消费消息
- 或者偏移量策略设置为**earliest**

## Unexpected handshake request with client mechanism PLAIN, enabled mechanisms are []

Kafka配置时出现`IllegalSaslStateException`，客户端使用了不被集群支持的PLAIN认证机制。

原因：**Kafka集群未配置认证，而代码使用了认证。**

解决方法：

1. 修改代码以**移除认证配置**
2. 或者，Kafka集群添加PLAIN认证支持