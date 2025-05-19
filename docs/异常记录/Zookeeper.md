# Zookeeper 异常记录

## KeeperException$ConnectionLossException

客户端连接zookeeper报错：`org.apache.zookeeper.KeeperException$ConnectionLossException`

原因：客户端与服务器之间的连接问题

解决：

1. 网络连接

2. 确认ZooKeeper服务器状态

3. 确认客户端配置，例如：sessionTimeout

   1. 在zookeeper的log目录下查看日志文件，看是否有显示超时的日志

   2. tail -n100 .out文件

   3. ```
      Unable to read additional data from client sessionid 0x0, likely client has closed socket
      ```

4. 验证版本兼容性

5. 重连机制：连接异常，重连其它节点

## Apache Curator 版本兼容

`Apache Curator` 是一个高层次的 `Zookeeper` **客户端库。**

以下是一些常见的 Curator 和 Zookeeper 版本对应关系：

1. Curator 5.x:
   - 通常与 Zookeeper 3.5.x 和 3.6.x 兼容。
   - 具体版本可能会有细微差异，建议查看 Curator 的发行说明。
2. Curator 4.x:
   - 主要与 Zookeeper 3.4.x 和 3.5.x 兼容。
   - Curator 4.x 开始支持 Zookeeper 3.5.x 的动态重配置和其他新特性。
3. Curator 3.x:
   - 主要与 Zookeeper 3.4.x 兼容。
   - 这个版本是为 Zookeeper 3.4.x 设计的，使用较老的 API。

### 确认兼容性

- **Apache Curator GitHub 仓库**：每个版本的发布说明中通常会提到兼容的 Zookeeper 版本。
- **Maven Central**：查看具体的 Curator 版本的 POM 文件，里面可能会列出依赖的 Zookeeper 版本。
- **官方文档**：Curator 和 Zookeeper 的官方文档中通常会提到兼容性信息。

### 选择合适的版本

在选择版本时，考虑以下几点：

- **项目需求**：如果需要使用 Zookeeper 的新特性，如动态重配置等，选择支持这些特性的 Curator 版本。
- **稳定性**：优先选择稳定版本，尤其是在生产环境中。
- **社区支持**：选择活跃维护的版本，以便获得及时的 bug 修复和安全更新。