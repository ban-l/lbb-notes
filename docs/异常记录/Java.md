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