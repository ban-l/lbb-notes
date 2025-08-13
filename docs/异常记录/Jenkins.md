## DownloadFailedException

```
org.owasp.dependencycheck.utils.DownloadFailedException: Error downloading file https://raw.githubusercontent.com/Retirejs/retire.js/master/repository/jsrepository.json; unable to connect.
```

**原因：**

由于 dependency-check-maven 插件在尝试下载漏洞数据库时网络连接失败导致的。

**解决：**

1. **配置代理（如果在代理环境下）**

   - 在 pom.xml 中为插件添加代理配置：

     ```xml
     <plugin>
         <groupId>org.owasp</groupId>
         <artifactId>dependency-check-maven</artifactId>
         <version>6.5.0</version>
         <configuration>
             <autoUpdate>true</autoUpdate>
             <!-- 如果使用代理，添加以下配置 -->
             <proxyUrl>http://your-proxy:port</proxyUrl>
             <proxyUser>username</proxyUser> <!-- 如果需要认证 -->
             <proxyPassword>password</proxyPassword> <!-- 如果需要认证 -->
         </configuration>
         <executions>
             <execution>
                 <goals>
                     <goal>check</goal>
                 </goals>
             </execution>
         </executions>
     </plugin>
     ```


2. **使用本地缓存或离线模式**

   ```xml
   <configuration>
       <autoUpdate>false</autoUpdate>
       <!-- 使用本地已有的数据库 -->
       <dataDirectory>/path/to/local/database</dataDirectory>
   </configuration>
   ```


3. **跳过漏洞检查**

   - 临时跳过漏洞检查：

     ```bash
     mvn clean install -DskipDependencyCheck
     ```

   - 或者在插件配置中设置：

     ```xml
     <configuration>
         <failBuildOnAnyVulnerability>false</failBuildOnAnyVulnerability>
     </configuration>
     ```


4. **更新插件版本**

   - 将 `dependency-check-maven` 插件更新到最新版本，可能已经修复了相关网络问题：

     ```xml
     <plugin>
         <groupId>org.owasp</groupId>
         <artifactId>dependency-check-maven</artifactId>
         <version>8.0.0</version> <!-- 使用较新版本 -->
         ...
     </plugin>
     ```


5. **配置镜像源**
   - 如果是网络访问限制问题，可以考虑使用国内镜像或者配置本地NVD镜像。