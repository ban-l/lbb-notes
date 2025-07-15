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

------

## MapStruct注意事项

使用`MapStruct`用于POJO之间的相互转换时，若源类含有`getXXX`形式的方法，没有`XXX`字段，则会报错。

所以使用时，注意方法命名。