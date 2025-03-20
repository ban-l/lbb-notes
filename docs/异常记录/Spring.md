## log4j12和logback 冲突

**日志**

```log
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/var/app/ocm-manager.jar!/BOOT-INF/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/var/app/ocm-manager.jar!/BOOT-INF/lib/logback-classic-1.2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Exception in thread "main" java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
        at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: java.lang.IllegalArgumentException: LoggerFactory is not a Logback LoggerContext but Logback is on the classpath. Either remove Logback or the competing implementation (class org.slf4j.impl.Log4jLoggerFactory loaded from jar:file:/var/app/ocm-manager.jar!/BOOT-INF/lib/slf4j-log4j12-1.7.25.jar!/). If you are using WebLogic you will need to add 'org.slf4j' to prefer-application-packages in WEB-INF/weblogic.xml: org.slf4j.impl.Log4jLoggerFactory
        at org.springframework.util.Assert.instanceCheckFailed(Assert.java:655)
        at org.springframework.util.Assert.isInstanceOf(Assert.java:555)
```

**原因**：项目中同时存在Logback和Log4j 1.x的SLF4J绑定，导致SLF4J无法确定使用哪个日志框架。

**解决**：**保留Logback（推荐）**

移除冲突的 `slf4j-log4j12` 依赖，保持Spring Boot默认的Logback。

1. **查找依赖来源**：使用Maven分析依赖树，找到引入 `slf4j-log4j12` 的库：
   
   ```bash
   mvn dependency:tree | grep slf4j-log4j12
   ```

2. **排除冲突依赖**：在对应的依赖项中添加排除：
   
   ```xml
   <dependency>
       <groupId>your.dependency.group</groupId>
       <artifactId>your-dependency-artifact</artifactId>
       <version>your-version</version>
       <exclusions>
           <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

3. **验证依赖**：重新构建项目并检查 `slf4j-log4j12` 是否已移除。

## Swagger2 - 版本冲突:documentationPluginsBootstrapper

版本冲突导致项目**启动报错**

```java
2024-07-29 10:23:24.385 ERROR 40632 --- [main] o.s.boot.SpringApplication : Application run failed
org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException
```

### 解决

#### `application.properties`配置

解决`SpringBoot`和`Swagger`版本冲突的问题

```
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

#### `SpringMVC`配置（可选）

同时配置**静态资源和`swagger`**，否则可能**映射**`swagger`**失败**。

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Overrde
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 静态资源配置
        registry.addResourceHandler("/**").
                addResourceLocations("classpath:/static/");
        // swagger配置
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
    }
}
```

### Swagger2 - Security配置

`SpringBoot`项目若集成了`Spring Security`，如果不做额外配置，`Swagger2`文档可能**会被拦截**，此时只需要在 `Spring Security` **配置类**中**重写** `configure`方法，添加如下过滤即可：

```java
@Override
public void configure(WebSecurity web) throws Exception {
   web.ignoring()
           .antMatchers("/swagger-ui.html")
           .antMatchers("/v2/**")
           .antMatchers("/swagger-resources/**");
}
```
