## IDEA导入多个module，无法识别子module

解决

1. Maven → Add Maven Projects
2. 重新导入子Maven，即单独选中子模块，而不是通过父pom.xml导入
   1. File → Project Structure → Modules → 点击“+” → Import Model
   2. 选择子模块下的 pom.xml 文件导入即可

## 工程突然找不到或引用不到jar包

原因：IDEA缓存造成。

- 解决：File -> Invalidate Caches，选择Invalidate and Restart 。

原因：修改pom文件造成。

- 解决：Reload project。