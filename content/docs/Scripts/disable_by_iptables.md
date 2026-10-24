+++
title = "修改iptables"
weight = 4
+++

```bash
# 需求，安全部门发现某台机器开通了不应该对外的端口： 例如 9100，9256，8080，38080
# 只针对 127.0.0.1 放开特定端口

#!/bin/env bash

# set iptables tmp
sudo iptables -I DOCKER-USER ! -s 127.0.0.1/32 -p tcp -m multiport --dports 9100,9256,8080,38080 -j DROP

# change iptables file
# set DOCKER-USER default
sudo sed '/^\*filter$/{
n
a\
:DOCKER-USER - [0:0]
}' /etc/sysconfig/iptables

# set DOCKER-USER rules
# get number
n=$(($(sudo grep -n "*security" /etc/sysconfig/iptables|cut -d: -f1) -1))
sudo sed "${n}i\#2026-10-02 封堵监控端口\\n-A DOCKER-USER ! -s 127.0.0.1/32 -p tcp -m multiport --dports 9100,9256,8080,38080 -j DROP\\n" /etc/sysconfig/iptables

```