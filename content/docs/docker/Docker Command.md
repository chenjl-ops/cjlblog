 +++
title = "Docker Command"
weight = 2
description="Docker 常用命令，用于备忘"
+++

```bash
# 本地查看image
docker images

# 拉去镜像(可指定版本,指定仓库)
docker pull nginx
#ep: docker pull nginx:1.27
#ep: docker pull registry.example.com/myapp:v1.0

# 查看镜像详情
docker inspect nginx

#按Dockerfile进行build(注意: . 为必须值)
docker build -t imageName:tag . -f Dockerfile
#ep:docker build -t chj_ecm_api_alpine:1.1 . -f Dockerfile

#远程仓库和本地仓库关联(tag之间关联)
docker tag imageName:tag 远程imageName:tag
#ep：docker tag chj_ecm_api_alpine:1.0 reg.chehejia.com/ds/cloud/devops/op-ecm-api:1.0

#本地image推送远程仓库(注意: 远程imageName为全路径)
docker push 远程imageName:tag
#ep：docker push reg.chehejia.com/ds/cloud/devops/op-ecm-api:1.0

#docker 启动服务
docker run -d --name 服务名称 --network 网络名称 --restart on-failure -e RUNTIME_ENV=test 远程imageName:tag
#ep：docker run -d --name go-sso-api --network host --restart on-failure -e RUNTIME_ENV=test -e RUNTIME_APP_NAME=go-sso-api -e RUNTIME_CONFIG_URL=http://10.2.0.167 harbor-public.123go.club/public-test/go-sso-api:v4
```