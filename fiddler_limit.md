# Fiddler 弱网模拟环境话术

Fiddler 是一款非常流行并且实用的http抓包工具，它的原理是在本机开启了一个http的代理服务器，然后它会转发所有的http请求和响应，有点像我们租房不直接跟房东租，而是通过中介进行交易，这样中介就可以把客户端(租户)与服务端(房东)的具体信息都收集到了，也就是我们俗称的“抓包”，

# fiddler模拟限速原理：

可以通过fiddler来模拟限速，因为fiddler本来就是一个代理服务器，它提供了客户端请求前和服务响应前的回调接口，可以在接口自定义逻辑。fiddler的模拟限速正是在客户端请求前来自定义限速的逻辑，次逻辑是通过延迟发送数据或接收数据的时间来限制网络的下载速度和上传速度，从而达到限速的效果。fiddler提供了功能，模拟低速网络环境，启动步骤如下：

Rules-->Performance-->Simulate  Modem  Speeds

勾选Simulate  Modem  Speeds，执行操作的时候，网络变慢了很多

自定义 Modem Speeds，其步骤如下：

1、点击菜单（Rules）

2、选择选项（Customize Rules），会弹出一个文档

找到下面的代码:

```
delay sends by 300ms per kb uploaded # 上行

delay sends by 300ms per kb downloaded # 下行
```

以上的值可自定义

上行可以理解为上传的速度，下行可以理解为下载的速度，举个例子，微信聊天，如果抓包做限速的话，上行就是发送消息给别人的速度，下行就是接收别人消息的速度。

这就是模拟模拟网络速度的原理，每上传/下载1kb要delay（延迟）多久。如果用kbps计算，其公式为：1000/下载速度=需要delay的时间（ms），比如50kb/s，则需要delay200ms来接收数据。

具体弱网环境指标参照：

![](./img/netdelay.png)