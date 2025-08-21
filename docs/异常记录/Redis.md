## unable to unfreeze entry

**问题：**

springboot集成Redisson，采用Jedis客户端，redisson版本是3.17.6，redis主从切换后，springboot应用程序报错：unable to unfreeze entry...... 

**原因：**

1. **拓扑更新延迟**
   Redisson 3.17.6 在 Redis 主从切换时，可能因以下原因无法刷新节点信息：
   - 未启用自动拓扑刷新机制，客户端缓存了旧的主节点地址13。
   - Jedis 虽比 Lettuce 恢复更快（约 15 秒），但在频繁切换时仍可能超时29。
2. **连接池冻结**
   报错 `unable to unfreeze entry` 表明 Redisson 尝试解冻被标记为“不可用”的节点连接池失败，通常因旧主节点恢复后角色冲突引发13。

**解决：**

3.17.6 存在主从切换处理的缺陷，高版本版已优化拓扑感知逻辑。

版本要在3.18.0及以上

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.33.0</version> <!-- 修复拓扑更新问题:cite[1]:cite[3] -->
</dependency>
```

------

## RedisCommandExecutionException

```lua
io.lettuce.core.RedisCommandExecutionException: 
ERR Error running script (): 
@enable_strict_lua:8: 
user_script:1: 
Script attempted to create global variable 'token'
```

**问题描述**

- **Redis在严格模式下不允许创建全局变量**

**解决**

- 使用局部变量`local`