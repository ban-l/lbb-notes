# druid连接耗尽

```log
Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 60000, active 1, maxActive 20, creating 1, ......
```

---

## 问题分析

**表面现象**

1. 连接池最大活动连接数 `maxActive=20`，但当前仅占用 `active=1`，理论上应有充足空闲连接。

2. 但应用仍报错“创建连接超时”（`wait millis=60000`），表明连接池无法正常分配连接。

**深层原因推测**

1. **连接泄漏**：某些连接未正确释放（如未关闭`Connection`/`Statement`/`ResultSet`），导致连接池中实际可用连接远低于预期。
2. **事务管理异常**：事务未正确提交/回滚，导致连接被长期占用（需检查事务边界）。
3. **数据库性能瓶颈**：数据库响应慢或最大连接数限制（如MySQL的`max_connections`参数过低）。
4. **网络问题**：数据库连接超时或不稳定。

---

## 解决方案

### 1. 排查连接泄漏

**启用连接池监控**

若使用Druid，开启监控功能，检查活跃连接数、等待线程数及泄漏检测（配置`removeAbandoned=true`，设置`logAbandoned=true`）。

```properties
# 示例Druid配置
spring.datasource.druid.remove-abandoned=true
spring.datasource.druid.log-abandoned=true
spring.datasource.druid.remove-abandoned-timeout=60
```

**代码审查**
检查`CallDao.queryCallList`方法及调用链，确保所有数据库资源在使用后关闭：

```java
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {
    // 处理结果集
} // 自动关闭资源
```

### 2. 验证事务管理

**检查事务边界**

- 确保事务方法（如`@Transactional`）正确配置，避免长事务或事务未提交导致连接占用。

**事务超时设置**

- 限制事务执行时间，防止长时间占用连接：

```java
@Transactional(timeout = 30) // 设置超时时间为30秒
public List<Call> queryCallList() { ... }
```

### 3. 优化连接池配置

**调整关键参数**

根据负载情况适当增加`maxActive`（如调整为50），并设置合理的`maxWait`（如30000ms）：

```properties
spring.datasource.max-active=50
spring.datasource.max-wait=30000
```

**启用连接有效性检测**

定期验证空闲连接可用性，避免池中存在无效连接：

```properties
spring.datasource.test-while-idle=true
spring.datasource.validation-query=SELECT 1
```

### 4. 检查数据库状态

**确认数据库最大连接数**

登录数据库查看当前连接数及上限（以MySQL为例）：

```sql
SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Threads_connected';
```

**优化慢查询**

分析`CallDao.queryCallList`对应的SQL性能，添加索引或优化查询逻辑。

---

## 验证步骤

1. 部署修复后，通过压力测试工具（如JMeter）模拟高并发请求，观察连接池使用情况。
2. 监控日志中是否仍出现连接超时异常。
3. 使用Druid监控页面实时查看连接分配状态。

---

## 总结

此问题本质是连接资源未正确释放或配置不当导致。

优先排查代码中的资源泄漏，其次优化连接池参数和数据库性能，最终确保系统在高负载下的稳定性。