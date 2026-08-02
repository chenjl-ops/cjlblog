+++
title = "Hello World"
weight = 1
description = "这是第一篇博客，主要测试一些基础功能"

[params]
math = true
+++

#### 这是第一篇博客，主要用于验证markdown


##### 这是mermaid验证
```mermaid

graph LR

A[用户]

B[Hugo]

C[Cloudflare]

A --> B --> C

```

```mermaid
flowchart TD
    subgraph APP["应用层"]
        A[Gateway]
        B[App-1]
        C[App-2]
        D[App-3]
        A --> B
        A --> C
        A --> D
    end
    
    subgraph MID["数据层"]
        DA[DB-MySQL]
        DB[MQ]
        DC[Redis]
    end
    
    subgraph WALLET["钱包层"]
        WA[Gateway]
        WB[wallet-app]
        WC[wallet-other]
        WA --> WB
        WA --> WC
    end

    APP --> MID
    WALLET --> MID
    

    
```
###### 这是上下关系
```mermaid
flowchart TD 
    A[开始] 
    B[处理] 
    C[结束] 
    
    A --> B 
    B --> C
```

###### 父子树节点
```mermaid
flowchart TD 
    CEO[CEO]  
    CTO[CTO] 
    CFO[CFO] 
    Dev1[开发A] 
    Dev2[开发B] 
    
    CEO --> CTO 
    CEO --> CFO 
    CTO --> Dev1 
    CTO --> Dev2

```

###### 类继承
```mermaid
classDiagram
    Animal <|-- Dog
    Animal <|-- Cat
```

######  ER图 一对多
```mermaid
erDiagram 
    DEPARTMENT ||--o{ EMPLOYEE : has

```


##### 数学公式验证
$$
E=mc^2
$$

$$
\int_0^\infty e^{-x^2} dx
$$