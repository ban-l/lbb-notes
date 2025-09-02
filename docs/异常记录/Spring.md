## JedisConnectionFactory was destroyed and cannot be used anymore

**错误信息：**

```
02/11:15:27.123 WARN (AnnotationConfi-main      :633) - Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'timeWheelTriggerController': Invocation of init method failed
02/11:15:27.141 ERROR(SpringApplicati-main      :859) - Application run failed

java.lang.IllegalStateException: JedisConnectionFactory was destroyed and cannot be used anymore
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.assertInitialized(JedisConnectionFactory.java:1061)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.getConnection(JedisConnectionFactory.java:880)
	at org.springframework.data.redis.core.RedisConnectionUtils.fetchConnection(RedisConnectionUtils.java:195)
	at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:144)
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:105)
	at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:383)
	at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:363)
	at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:350)
	at org.springframework.data.redis.stream.DefaultStreamMessageListenerContainer.lambda$getReadFunction$4(DefaultStreamMessageListenerContainer.java:235)
	at org.springframework.data.redis.stream.StreamPollTask.readRecords(StreamPollTask.java:146)
	at org.springframework.data.redis.stream.StreamPollTask.doLoop(StreamPollTask.java:127)
	at org.springframework.data.redis.stream.StreamPollTask.run(StreamPollTask.java:112)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:840)
```

**错误分析：**

应用程序（通常是 Spring 应用上下文）正在尝试使用一个已经被销毁（destroyed）的 `JedisConnectionFactory` Bean。

在 Spring 中，当一个 Bean 被销毁后（例如在应用关闭时），它所占用的资源（如网络连接、线程池等）会被清理，任何试图再次使用它的操作都会抛出 `IllegalStateException`。

1. **主要问题**：`timeWheelTriggerController` Bean 初始化失败导致整个应用启动失败。
2. **连锁反应**：应用启动失败触发 Spring 上下文销毁流程，`JedisConnectionFactory` 被销毁。
3. **最终表现**：Redis Stream 监听线程仍在运行并尝试使用已被销毁的连接工厂，导致 `IllegalStateException`。

**详细错误堆栈解读：**

1. `timeWheelTriggerController` Bean 初始化失败导致整个应用启动失败，触发 Spring 上下文销毁流程。
2. Redis Stream 监听线程仍在运行并尝试使用已被销毁的连接工厂，导致 `IllegalStateException`。
3. **触发点 (at ...JedisConnectionFactory.assertInitialized:1061):**
   `JedisConnectionFactory` 内部有一个 `destroyed` 状态标志。在调用任何实际操作（如 `getConnection()`）之前，它会调用 `assertInitialized()` 方法来检查自身状态。如果发现 `destroyed` 为 `true`，就立即抛出这个 `IllegalStateException`。
4. **调用链 (getConnection -> RedisConnectionUtils.getConnection -> RedisTemplate.execute -> ...):**
   堆栈显示，错误源于一个 `RedisTemplate.execute(...)` 的调用，其目的是为了执行一个 Redis 操作（这里是 Stream 相关的监听任务 `DefaultStreamMessageListenerContainer`）。
5. **执行上下文 (at ...ThreadPoolExecutor.runWorker...Thread.run):**
   最关键的信息是，这个操作是在一个**子线程**（由 `ThreadPoolExecutor` 管理）中运行的。这说明问题发生在**多线程环境**下。

**根本原因：应用关闭过程中的生命周期竞争条件（Race Condition）。**

1. 应用使用了 Spring Data Redis 的 Stream 监听功能（`@EnableRedisStreams`, `StreamMessageListenerContainer` 等），它会启动后台线程（如堆栈中的 `ThreadPoolExecutor`）来持续轮询 Redis 消息。
2. 当应用开始关闭时，Spring 的 `ApplicationContext` 开始执行销毁流程。
3. 销毁流程是有顺序的。Spring 会先发送上下文关闭事件，然后**单线程地**、按照依赖关系倒序销毁所有的 Bean。
4. **`JedisConnectionFactory` 被先销毁了**（将其状态标记为 `destroyed` 并关闭连接池）。
5. 然而，那个用于 Stream 监听的后台线程**还没有被及时关闭或中断**，它仍然在工作，试图从 Redis 获取新消息。当它下一次调用 `redisTemplate.execute()` 时，就会向已经被销毁的 `JedisConnectionFactory` 请求获取连接，从而抛出上述异常。

**解决方案：**正确配置 StreamMessageListenerContainer 的生命周期，程序启动失败时，关闭streamContainer。

```java
@Component
@RequiredArgsConstructor
public class ElegantExit implements ApplicationListener<ApplicationFailedEvent> {

    private final StreamMessageListenerContainer<String, ObjectRecord<String, String>> streamContainer;

    @Override
    public void onApplicationEvent(ApplicationFailedEvent event) {
        log.warn("程序启动失败，关闭streamContainer");
        if (streamContainer != null && streamContainer.isRunning()) {
            try {
                streamContainer.stop();
            } catch (Exception e) {
                log.error("关闭streamContainer时发生异常", e);
            }
        }
    }

}
```

------

## maven install (repackage failed: Unable to find main class）

背景

- 自定义starter打成jar包，但`maven install`时却提示缺少主类，但这是一个公共模块，无主类。

原因

- 使用spring boot项目，用的maven插件为`spring-boot-maven-plugin`

- 用此插件打包时，会默认寻找签名是`public static void main(String[] args)`的方法，没有所以报错

**解决1：修改配置，`<skip>true</skip>`**

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

**解决2：改用apache的maven插件**

```xml
 <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
</plugin>
```

------

## An attempt was made to call a method that does not exist. 

`An attempt was made to call a method that does not exist. The attempt was made from the following location:`

依赖版本冲突，可能更新了依赖，不兼容之前的配置

------

## 循环依赖问题

**问题描述**

```
Description:

The dependencies of some of the beans in the application context form a cycle:

   kafkaConfig
      ↓
   backContainerImpl
      ↓
   resultConsumerFactory
┌─────┐
|  timeWheelOperations defined in class path resource [cc/wellcloud/framework/timewheel/spring/boot/autoconfigure/config/TimeWheelAutoConfiguration.class]
↑     ↓
|  redisTimeWheel defined in class path resource [cc/wellcloud/framework/timewheel/spring/boot/autoconfigure/config/TimeWheelAutoConfiguration.class]
↑     ↓
|  customDelayTaskListener defined in file [D:\\workspace-wellcloud\\custom-module\\target\\classes\\cc\\wellcloud\\ocm\\timewheel\\CustomDelayTaskListener.class]
└─────┘
```

**解决**

1. 单例模式下的`setter`循环依赖
2. 使用 `@Lazy` 注解
3. 使用 `@Configuration `和 `@Bean` 显式地定义Bean的创建顺序。
4. 使用 `@Lookup` 注解解决循环依赖

------

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

------

## Swagger2 - 版本冲突:documentationPluginsBootstrapper

版本冲突导致项目**启动报错**

```java
2024-07-29 10:23:24.385 ERROR 40632 --- [main] o.s.boot.SpringApplication : Application run failed
org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException
```

**解决**

**`application.properties`配置**

```
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

**`SpringMVC`配置（可选）**

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

**Swagger2 - Security配置**

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

------

## org.apache.commons.pool2.impl.GenericObjectPoolConfig.setMaxWait(Ljava/time/Duration;)

**原因：**

`spring-boot-starter-parent`和`spring-data-redis(commons-pool2)`版本冲突

**解决：**

1. 单独升级`commons-pool2`版本
2. 降低`spring-boot`版本

------

## jar运行报错no main manifest attribute

**原因：找不到主类。**

一般情况下，`java`打包成 `jar`包需要在`MANIFEST.MF` 中指定 `Start-Class`项，以便运行 `java -jar xxx.jar` 时找到对应的**主类**。

查看`META-INF`下的`MANIFEST.MF`，正常的文件如下：

```
Manifest-Version: 1.0
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Archiver-Version: Plexus Archiver
Built-By: Ban
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Start-Class: cc.wellcloud.cloud.CallbackSupplementApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Version: 2.7.18
Created-By: Apache Maven 3.8.1
Build-Jdk: 1.8.0_261
Main-Class: org.springframework.boot.loader.JarLauncher
```

**解决：修改`POM`文件中的打包插件配置**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
          	......
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>${spring-boot.version}</version>
            <configuration>
		            <!-- 主类 -->
                <mainClass>cc.wellcloud.cloud.CallbackSupplementApplication</mainClass>
            </configuration>
            <executions>
                <execution>
                    <id>repackage</id>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
    <!-- 置打包后的jar包名称 -->
    <finalName>callback-supplement</finalName>
</build>
```

