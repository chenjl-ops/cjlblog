 +++
title = "DockerFile Example"
weight = 2
description="dockerfile一些例子"
+++

```docker
# Python36 + Django 2.1.3

FROM python:3.6.8 As cmdb-web-base
WORKDIR /chj/app/op-cmdb-web
 
RUN pip install --upgrade pip
 
COPY requirements.txt ./
RUN pip install -r requirements.txt
 
FROM python:3.6.8-alpine3.8
WORKDIR /chj/app/op-cmdb-web
COPY --from=cmdb-web-base /usr/local/lib/python3.6/site-packages /usr/local/lib/python3.6/site-packages
COPY ./assets_manage .
EXPOSE 8003
CMD ["python", "manage.py", "runserver", "0.0.0.0:8003"]
```

```docker
# FastApi + Python3

FROM reg.chehejia.com/chj-k8s-change/public/python:3.6.8_20200103 As builder

WORKDIR /chj/app/{{ packageName }}
COPY requirements.txt .
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt

FROM python:3.6.8-alpine3.8

COPY --from=builder /usr/local/lib/python3.6/site-packages /usr/local/lib/python3.6/site-packages

Set System TimeZone
RUN apk add --no-cache tzdata bash
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/TZ /etc/localtime && echoTZ > /etc/timezone

RUN mkdir -p /chj/app && mkdir -p /chj/data/log/{{ packageName }} && chmod 775 -R /chj

WORKDIR /chj/app/{{ packageName }}
COPY . .
CMD uvicorn main:app --host 0.0.0.0 --port ${NGINX_PORT}
```

```docker
# vue ui

FROM node:11.7.0-alpine as base-build
WORKDIR /chj/app/op-cmdb-ui
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
 
FROM nginx:1.14.2-alpine as production-stage
COPY --from=base-build /chj/app/op-cmdb-ui/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```docker
## golang 项目 gin

# Build stage
FROM golang:1.22.4 AS builder
#ENV GOPROXY=https://proxy.golang.com.cn

WORKDIR /app

COPY . .

#RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o ./bin/main ./cmd/app
RUN bash ./scripts/build.sh

# Production stage
FROM amd64/alpine:latest

# Set System TimeZone
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV WORKDIR /app
WORKDIR /app

COPY --from=builder /app/bin/main .

CMD ./main start
```