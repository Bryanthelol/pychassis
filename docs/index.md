---
#search:
#  boost: 2 
#search:
#  exclude: true
hide:
  - footer
---

# 概述

## Nameko

在分布式环境下，Nameko 提供了一种微服务架构方式，使得微服务间的通信变得简单，调用一个服务就像调用了一个本地方法一样。

Nameko 提供 RPC 同步调用（同时支持异步调用），事件发布/订阅，以及暴露 HTTP 协议的 API（但一般不建议使用，而是使用其他 web 框架）。

以上这一切能力都通过基于 AMQP 协议的 RabbitMQ 之上达成。

当你启动一个 Nameko 微服务进程，服务的发现已经通过 RabbitMQ 完成，所以对于开发者来说，接下来只需要关注业务逻辑的实现即可。

如果业务量加大，对微服务的请求数量上升到了一定程度，那么只需要增加 Nameko 微服务的实例数量就可以轻松应对，底层的 RabbitMQ 会自动对 Nameko 实例做负载均衡。

## 命令行工具 pychassiscli

pychassiscli 是一个命令行工具，开发人员本地编写项目代码时使用。

安装 `pip install pychassiscli`

提供功能（命令详细信息使用 `pycli --help` 查看）：

- 生成项目模版文件，项目脚手架
	- `pycli gen -d <dir_name> -t nameko`
  
- 操作 metrics 服务
    - 生成 metrics 配置文件
		- `pycli gen -d <dir_name> -t metrics`
	- 启动 metrics 服务
		- `pycli start -s metrics`
	- 停止 metrics 服务
		- `pycli stop -s metrics`
	- 刷新 metrics 服务的配置
		- `pycli refresh -s metrics `

## 公共代码和包依赖集合库 pychassislib

pychassislib 是一个应用运行的依赖库，测试或生产环境使用。


安装 `pip install pychassislib`

提供的公共代码和依赖项见源码：https://github.com/Bryanthelol/pychassislib


