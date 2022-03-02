---
title: "Clean Architecture on Golang"
date: 2022-02-02
tags: ['golang', '后端']
---

<https://medium.com/hackernoon/golang-clean-archithecture-efd6d7c43047>

文章中的架构主要分为四个部分：Model, Repository, Usecase, Delivery

## Model

Model 指得是业务实体，在 Golang 中可以是一个 Struct 其中有业务需要的数据类型

```golang
import "time"

type Article struct {
    ID        int64     `json:"id"`
    Title     string    `json:"title"`
    Content   string    `json:"content"`
    UpdatedAt time.Time `json:"updated_at"`
    CreatedAt time.Time `json:"created_at"`
}
```

## Repository

Repository 主要负责数据库的操作，作用就是不让业务逻辑部分直接操作数据库，构造 sql query 是在这一层中实现的

## Usecase

业务逻辑部分，决定如何调用 Repository 中的操作和向 Delivery 层传递数据

## Delivery

主要就是与外界的通讯层，比如 http 连接或 rpc 请求，这也是 web api 框架作用的层级
