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

