## `tk.mybatis` 插入失败，主键重复

**背景**

数据库：id自增，当前`AUTO_INCREMENT=1411`。

代码如下：

```java
// Mapper接口
@org.apache.ibatis.annotations.Mapper
public interface UserMapper extends Mapper<User> {
}

// 插入操作
public void save(User user) {
    userMapper.insertSelective(user);
}
```

**插入失败，主键重复**

第一次插入，插入成功。

1. 对象`id`回写为0。
2. 数据插入数据库，`id`为1411。
3. `AUTO_INCREMENT=1412`

```
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Preparing: INSERT INTO user ( id,name,age ) VALUES( ?,?,? )
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Parameters: 0(Long), 小明(String), 18(Integer)
DEBUG(insertSelective-XNIO-1 tas:135) - <==  Updates: 1
```

第二次插入失败，报错主键重复。

1. 对象`id`回写为1411。
2. 数据库已存在`id`为1411的数据，插入失败。

```
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Preparing: INSERT INTO user ( id,name,age ) VALUES( ?,?,? )
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Parameters: 1411(Long), 小明2(String), 19(Integer)
ERROR(SessionConcurre-XNIO-1 tas:122) - 插入用户失败
```

第三次插入失败，报错主键重复。

1. 对象`id`回写为1411。
2. 数据库已存在`id`为1411的数据，插入失败。

```
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Preparing: INSERT INTO user ( id,name,age ) VALUES( ?,?,? )
DEBUG(insertSelective-XNIO-1 tas:135) - ==>  Parameters: 1411(Long), 小明3(String), 20(Integer)
ERROR(SessionConcurre-XNIO-1 tas:122) - 插入用户失败
```

后续插入操作均失败......

**原因：`@org.apache.ibatis.annotations.Mapper`与`tk.mybatis`冲突。**

- **重复注册**：`@Mapper`注解会注册接口，而`tk.mybatis`的`@MapperScan`也会注册它，导致注册两次。
- **功能覆盖**：MyBatis 官方的注册逻辑可能无法正确识别`tk.mybatis`的扩展功能（如主键回写），导致部分配置失效。

**解决方案一**

1. `Mapper`接口移除`@org.apache.ibatis.annotations.Mapper`注解。
2. 启动类/配置类加上`tk.mybatis` 的 `@MapperScan`。

**解决方案二**

主键字段加上`@Column(insertable = false)`，忽略id。

------

## factoryBeanObjectType

```
java.lang.IllegalArgumentException: 
Invalid value type for attribute 'factoryBeanObjectType': java.lang.String
```

原因：mybatis和springboot的版本不匹配

解决：springboot版本和mybatis版本适配，例如：springboot 3.2.2、mybatis 3.0.3

```xml
<!--springBoot--> 
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.2</version>
        <relativePath/> <!-- lookup parent from repository -->
 </parent>
 <!--MyBatis-->
 <dependency>
         <groupId>org.mybatis.spring.boot</groupId>
         <artifactId>mybatis-spring-boot-starter</artifactId>
         <version>3.0.3</version>
 </dependency>
```

------

##  "mapper" 必须匹配 DOCTYPE 根 "null”

`Caused by: org.xml.sax.SAXParseException`: 文档根元素 "mapper" 必须匹配 DOCTYPE 根 "null”

原因：mapper.xml缺少头部配置文件

解决

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="org.lbb.mapper.YourMapper">
</mapper>
```

------

## Caused by: org.xml.sax.SAXParseException: 元素内容必须由格式正确的字符数据或标记组成。

```
Caused by: org.apache.ibatis.builder.BuilderException: Error creating document instance.  Cause: org.xml.sax.SAXParseException; lineNumber: 29; columnNumber: 22; 元素内容必须由格式正确的字符数据或标记组成。
	at org.apache.ibatis.parsing.XPathParser.createDocument(XPathParser.java:262)
	at org.apache.ibatis.parsing.XPathParser.<init>(XPathParser.java:127)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.<init>(XMLMapperBuilder.java:83)
	at org.mybatis.spring.SqlSessionFactoryBean.buildSqlSessionFactory(SqlSessionFactoryBean.java:692)
	... 71 common frames omitted
Caused by: org.xml.sax.SAXParseException: 元素内容必须由格式正确的字符数据或标记组成。
```
原因：在 MyBatis 的 XML 配置中，会因为 XML 解析器对特殊字符的处理导致错误。
例如：对于比较运算符 `<` 解析错误

```mysql
SELECT * FROM t_user
WHERE create_dt < #{create_dt}  
```

解决：将 SQL 语句中的特殊符号用 `<![CDATA[ ]]>` 包裹起来，以防止解析器误解析。

```mysql
`<![CDATA[
  SELECT * FROM t_user
  WHERE create_dt < #{create_dt}
]]>


SELECT * FROM t_user
WHERE create_dt <![CDATA[<]]> #{create_dt}  
```

