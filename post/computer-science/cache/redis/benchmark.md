---
draft: false
date: 2022-05-13 08:00:00 +0800
lastmod: 2022-05-13 08:00:00 +0800
title: "redis benchmark"
summary: "redis benchmark"
toc: true

categories:
- redis

tags:
- computer-science(计算机科学)
- cache(缓存)
- redis
---

### 支持的参数

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

- `-h`：连接地址
- `-p`：连接端口
- `-d`：执行 SET/GET 的数据量（bytes）
- `-q`：每秒能处理的请求数

### 测试 SET/GET 在不同数据量下的性能

```
redis-benchmark -h 127.0.0.1 -p 6379 -q -d {bytes}
```

| bytes       | 10       | 20       | 50       | 100      | 200      | 1000     | 5000     | 10000    | 15000    |
| ---         | ---      | ---      | ---      | ---      | ---      | ---      | ---      | ---      | ---      |
| PING_INLINE | 54436.58 | 57504.31 | 58582.31 | 54229.93 | 54914.88 | 60204.70 | 59523.81 | 58309.04 | 57770.08 |
| PING_BULK   | 58719.91 | 57670.13 | 59276.82 | 55803.57 | 59594.76 | 60753.34 | 60642.81 | 60975.61 | 61652.28 |
| SET         | 59880.24 | 57570.52 | 59382.42 | 56657.22 | 58997.05 | 60532.69 | 58207.21 | 54824.56 | 51652.89 |
| GET         | 59417.71 | 58513.75 | 57339.45 | 55586.44 | 58105.75 | 62344.14 | 61957.87 | 59952.04 | 62189.05 |
| INCR        | 58823.53 | 59311.98 | 55555.56 | 58892.82 | 57670.13 | 62539.09 | 61652.28 | 61614.29 | 63051.70 |
| LPUSH       | 61996.28 | 59101.65 | 54975.26 | 56561.09 | 58038.30 | 59488.40 | 53879.31 | 54436.58 | 51124.75 |
| RPUSH       | 58616.65 | 59031.88 | 52938.06 | 56785.91 | 56433.41 | 59701.50 | 57208.24 | 53134.96 | 51599.59 |
| LPOP        | 59241.71 | 59844.41 | 57971.02 | 57504.31 | 55803.57 | 58892.82 | 61462.82 | 62421.97 | 62774.64 |
| RPOP        | 61881.19 | 57636.89 | 57208.24 | 57971.02 | 59665.87 | 59988.00 | 61804.70 | 61349.70 | 62344.14 |
| SADD        | 61919.50 | 57339.45 | 57208.24 | 56785.91 | 57971.02 | 60975.61 | 62617.41 | 57570.52 | 60901.34 |
| HSET        | 60204.70 | 60864.27 | 60204.70 | 58377.11 | 58479.53 | 60096.15 | 58858.15 | 51867.22 | 52056.22 |
| SPOP        | 58479.53 | 60168.47 | 60096.15 | 58927.52 | 59559.26 | 62893.08 | 60422.96 | 55679.29 | 63131.31 |
| LPUSH       | 59737.16 | 59241.71 | 54024.85 | 58582.31 | 60277.27 | 60569.35 | 54259.36 | 51894.13 | 52219.32 |
| LRANGE_100  | 61274.51 | 60422.96 | 54914.88 | 54884.74 | 60938.45 | 58275.06 | 60060.06 | 58892.82 | 59737.16 |
| LRANGE_300  | 57603.69 | 57208.24 | 54259.36 | 57273.77 | 60168.47 | 58072.01 | 60975.61 | 58377.11 | 61957.87 |
| LRANGE_500  | 58997.05 | 57537.40 | 54704.60 | 59382.42 | 61804.70 | 59594.76 | 60277.27 | 57175.53 | 59523.81 |
| LRANGE_600  | 60132.29 | 59772.86 | 56657.22 | 59171.59 | 59665.87 | 59844.41 | 60716.46 | 58445.36 | 60496.07 |

```chart
{
  "type": "line",
  "data": {
    "labels": [
      "10","20","50","100","200","1000","5000","10000","15000"
    ],
    "datasets": [
      {
        "label": "PING_INLINE",
        "data": [54436.58,57504.31,58582.31,54229.93,54914.88,60204.70,59523.81,58309.04,57770.08],
        "borderColor": "rgba(0,0,255,1)",
        "fill": false
      },
      {
        "label": "PING_BULK",
        "data": [58719.91,57670.13,59276.82,55803.57,59594.76,60753.34,60642.81,60975.61,61652.28],
        "borderColor": "rgba(0,0,255,1)",
        "fill": false
      },
      {
        "label": "SET",
        "data": [59880.24,57570.52,59382.42,56657.22,58997.05,60532.69,58207.21,54824.56,51652.89],
        "borderColor": "rgba(0,255,0,1)",
        "fill": false
      },
      {
        "label": "GET",
        "data": [59417.71,58513.75,57339.45,55586.44,58105.75,62344.14,61957.87,59952.04,62189.05],
        "borderColor": "rgba(0,255,0,1)",
        "fill": false
      },
      {
        "label": "INCR",
        "data": [58823.53,59311.98,55555.56,58892.82,57670.13,62539.09,61652.28,61614.29,63051.70],
        "borderColor": "rgba(0,255,255,1)",
        "fill": false
      },
      {
        "label": "LPUSH",
        "data": [61996.28,59101.65,54975.26,56561.09,58038.30,59488.40,53879.31,54436.58,51124.75],
        "borderColor": "rgba(255,255,0,1)",
        "fill": false
      },
      {
        "label": "RPUSH",
        "data": [58616.65,59031.88,52938.06,56785.91,56433.41,59701.50,57208.24,53134.96,51599.59],
        "borderColor": "rgba(255,255,0,1)",
        "fill": false
      },
      {
        "label": "LPOP",
        "data": [59241.71,59844.41,57971.02,57504.31,55803.57,58892.82,61462.82,62421.97,62774.64],
        "borderColor": "rgba(255,0,0,1)",
        "fill": false
      },
      {
        "label": "RPOP",
        "data": [61881.19,57636.89,57208.24,57971.02,59665.87,59988.00,61804.70,61349.70,62344.14],
        "borderColor": "rgba(255,0,0,1)",
        "fill": false
      },
      {
        "label": "SADD",
        "data": [61919.50,57339.45,57208.24,56785.91,57971.02,60975.61,62617.41,57570.52,60901.34],
        "borderColor": "rgba(160,32,240,1)",
        "fill": false
      },
      {
        "label": "HSET",
        "data": [60204.70,60864.27,60204.70,58377.11,58479.53,60096.15,58858.15,51867.22,52056.22],
        "borderColor": "rgba(255,165,0,1)",
        "fill": false
      },
      {
        "label": "SPOP",
        "data": [58479.53,60168.47,60096.15,58927.52,59559.26,62893.08,60422.96,55679.29,63131.31],
        "borderColor": "rgba(160,32,240,1)",
        "fill": false
      },
      {
        "label": "LPUSH",
        "data": [59737.16,59241.71,54024.85,58582.31,60277.27,60569.35,54259.36,51894.13,52219.32],
        "borderColor": "rgba(255,181,197,1)",
        "fill": false
      },
      {
        "label": "LRANGE_100",
        "data": [61274.51,60422.96,54914.88,54884.74,60938.45,58275.06,60060.06,58892.82,59737.16],
        "borderColor": "rgba(255,181,197,1)",
        "fill": false
      },
      {
        "label": "LRANGE_300",
        "data": [57603.69,57208.24,54259.36,57273.77,60168.47,58072.01,60975.61,58377.11,61957.87],
        "borderColor": "rgba(255,181,197,1)",
        "fill": false
      },
      {
        "label": "LRANGE_500",
        "data": [58997.05,57537.40,54704.60,59382.42,61804.70,59594.76,60277.27,57175.53,59523.81],
        "borderColor": "rgba(255,181,197,1)",
        "fill": false
      },
      {
        "label": "LRANGE_600",
        "data": [60132.29,59772.86,56657.22,59171.59,59665.87,59844.41,60716.46,58445.36,60496.07],
        "borderColor": "rgba(255,181,197,1)",
        "fill": false
      }
    ]
  }
}
```

从图表上看，从 1KB 开始，SET、HSET、LPUSH、RPUSH 有明显的下降趋势。

### 测试不同长度 value 占用内存的情况

本地使用 redis-cli 工具连接 redis 客户端，然后使用 `info memory` 命令，查看内存占用情况。

如果报错：`NOAUTH Authentication required.`。就先使用 `auth` 命令，输入密码。

下面的测试，key 始终是 `test_str_200000...test_str_400000`（15 byte）。

```
used_memory:763320
used_memory_human:745.43K
# 添加 20w 个：max int32
used_memory:15660512
used_memory_human:14.00M
```

```
used_memory:763472
used_memory_human:745.58K
# 添加 20w 个：10 个 a
used_memory:18860624
used_memory_human:17.00M
```

```
used_memory:763624
used_memory_human:745.73K
# 添加 20w 个：50 个 a
used_memory:26860816
used_memory_human:25.00M
```

```
used_memory:763776
used_memory_human:745.88K
# 添加 20w 个：100 个 a
used_memory:38060928
used_memory_human:36.00M
```

```
used_memory:763928
used_memory_human:746.02K
# 添加 20w 个：1000 个 a
used_memory:222558232
used_memory_human:212.00M
```

| time | data len | per len | ext len |
| ---  | ---      | ---     | ---     |
| 9.3s | 4b       | 74b     | 56b     |
| 9.4s | 10b      | 90b     | 65b     |
| 9.2s | 50b      | 130b    | 65b     |
| 9.0s | 100b     | 186b    | 71b     |
| 9.4s | 1000b    | 1108b   | 93b     |

$ext len（除去有效数据的大小） = per len（用内存信息计算出来的平均大小） - data len（数据的大小） - key len（key 的大小）$

从图表上看，value 变大时，会额外占用更多的空间。

### 参考

- [Redis benchmark](https://redis.io/docs/reference/optimization/benchmarks/)