+++
title = "aws vpc subnet"
description="以aws的一个vpc下有三个子网为例子"
+++
```
# 以一个vpc下有三个zone区为例：
# 例如 region：HK，Zone：a、b、c

# 公有子网，每个子网256个地址
subnet-public-a
10.1.0.0/24
subnet-public-b
10.1.1.0/24
subnet-public-c
10.1.2.0/24
# 预留 10.1.3.0/24 -- 10.1.31.0/24 个公有子网

# app私有子网，每个子网有8192个地址
subnet-private-app-a
10.1.32.0/19
subnet-private-app-b
10.1.64.0/19
subnet-private-app-c
10.1.96.0/19

# DB私有子网，每个子网有4096个地址
subnet-private-db-a
10.1.128.0/20
subnet-private-db-b
10.1.144.0/20
subnet-private-db-c
10.1.160.0/20

# 最终还预留5个/20子网，每个4096个地址
10.1.176.0/20
10.1.192.0/20
10.1.208.0/20
10.1.224.0/20
10.1.240.0/20

# 以及预留 10.1.241.0/24 -- 10.1.255.0/24 个地址

```