## redis

- 2021-01-26

### 连接问题

异常场景：redis密码正确，使用工具可以连接，但是程序连接不上去。

可行的解决方案：先在redis中增加一个用户，然后把auth参数变成username:password的格式。