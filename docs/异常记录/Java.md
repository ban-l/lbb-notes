## Optional int parameter '' is present but cannot be

```
Optional int parameter 'timeout' is present but cannot be translated into a null value 
due to being declared as a primitive type. 
Consider declaring it as object wrapper for the corresponding primitive type
```

**接口传参时**，存在`int`参数`timeout`，但由于被声明为基本类型，因此无法转换为空值。

考虑将其声明为对应基本类型的**对象包装器**。

------

## 切换网络导致的 connection timeout

**原因：IP地址变化**

解决

1. 插入网线和无线网的ip不同
2. 本地测试时，注意IP地址的变化 ，及时切换。
3. 尤其分布式调度时，注意IP地址的变化 ，防止连接超时。

------

## DataIntegrityViolationException

DataIntegrityViolationException通常表示违反了数据完整性约束，比如数据库中的唯一性约束、外键约束等。

通常发生在尝试Insert或Update数据时，与数据库表结构定义的约束条件不符。

请检查以下几点：
1. **唯一性约束**：确认你正在插入的数据是否已经存在。
2. **外键约束**：确保你引用的数据在关联表中确实存在。
3. **非空约束**：检查是否有字段被设置为 `NOT NULL` 但未提供值。
4. **长度限制**：确保字符串类型的数据没有超过字段定义的最大长度。


## 使用MessageFormat拼接字符串

使用`MessageFormat`拼接字符串，**参数类型为数字**时，会出现**逗号分隔**的问题，**导致key值错误**。

```java
 // 使用MessageFormat拼接字符串，参数类型为数字时，会出现逗号分隔的问题，导致key值错误。
 long tenantId = 123456789L;
 String caller = "test";

 String key1 = MessageFormat.format("yanxi:callback:queue:{0}:{1}", tenantId, caller);
 System.out.println(key1); // yanxi:callback:queue:123,456,789:test

 String key2 = MessageFormat.format("yanxi:callback:queue:{0}:{1}", tenantId+"", caller);
 System.out.println(key2); // yanxi:callback:queue:123456789:test
```
