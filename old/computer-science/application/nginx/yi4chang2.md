## 异常处理

两个lnmp环境交互。由于网络请求受阻，一段时间后，环境b访问环境a时产生如下报错。

```shell
upstream timed out (110: Connection timed out) while reading response header from upstream
```

推测原因是，环境a由于访问环境b的curl超时时间设置为30s，所以网络受阻后，大量的请求在curl阶段进入等待状态，但是这些请求依然在占用php-fpm的worker进程，刚开始等待中的请求数量还在环境a的可承受范围内，所以没有上述报错。

一段时间后阻塞的curl耗尽了环境A的php-fpm的worker进程，导致后续环境B访问环境A时，nginx分配不出worker进程，导致如上报错。