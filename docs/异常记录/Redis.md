## RedisCommandExecutionException

```lua
io.lettuce.core.RedisCommandExecutionException: 
ERR Error running script (): 
@enable_strict_lua:8: 
user_script:1: 
Script attempted to create global variable 'token'
```

**问题描述**

- **Redis在严格模式下不允许创建全局变量**

**解决**

- 使用局部变量`local`