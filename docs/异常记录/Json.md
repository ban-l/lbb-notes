## safeMode not support autoType : …

**原因**：fastjson开启了**安全模式**(safeMode)

`ParserConfig.getGlobalInstance().setSafeMode(true);`

**解决**

1. 建议开启：开启安全模式**禁止**自动类型转化(`@Type`)
2. **若使用，不做任何设置即可**

------

## com.fasterxml.jackson.databind.exc.InvalidDefinitionException

**问题描述**

```yaml
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: 
Cannot construct instance of 'xxx' (no Creators, like default construct, exist): 
cannot deserialize from Object value (no delegate- or property-based Creator)
```

**原因：**

`com.fasterxml.jackson.databind.ObjectMapper.readValue(String content, Class<T> valueType)`方法，传入的`class`对象**没有无参构造器**，具体原因是在该对象上同时使用了`@Data`和`@AllArgsConstructor`注解，`@AllArgsConstructor`**阻止**了`@Data`**生成（无参）构造器**，从而该对象只有一个全参构造器，没有无参构造器，导致反序列化失败。

**注意：**

- 显式构造器、 `@RequiredArgsConstructor`, `@AllArgsConstructor`都会**抑制`@Data`生成构造器**。

**解决：**

- 显式添加**无参构造器**
- 或者使用`@NoArgsConstructor`