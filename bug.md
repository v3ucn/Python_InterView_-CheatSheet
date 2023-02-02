##支付宝公钥和私钥遇到的问题

首先，新版支付宝应用了新的的加密算法，所以生成应用公钥和私钥的时候要使用RSA2的算法

之前我在请求支付接口的时候，误操作把应用公钥发送了过去，导致接口504，实际上发送给支付宝接口的私钥应该是我自己生成应用私钥，而公钥则应该使用支付宝的公钥，而不是我自己生成的应用公钥

##支付宝退款遇到的问题

在开发支付宝退款的业务时，支付宝官网有两个订单字段 out_trade_no和trade_no，我误以为都是订单号，所以随便传了一个导致退款错误，实际上out_trade_no是商户订单号，而trade_no则是支付宝自己生成的支付宝交易号，用这两个号都可以退换成功，但是out_trade_no是商户自己的订单号，本身也存在数据库里，而支付宝交易号如果在回调方法内不入库，则无法在退款的时候传递给支付宝，所以建议使用商户订单号 out_trade_no

##支付时遇到的性能问题

在支付宝支付功能上线后，发现在某些时间段内，服务器就会报警，查了一下，内存占用过高，理论上一个很普通的支付宝三方支付业务不应该会导致性能问题，仔细看了一下源代码，发现当时写支付类的时候很随意，没有考虑到每次支付都会在内存中新建实例，而python的内存管理机制导致支付结束后不会主动销毁，所以我改造了一下支付类，将其改造成了单例模式，这样就解决了支付频率过高导致内存占用过高的问题。

```
from functools import wraps
def singleton(cls):
    instance = None
    @wraps(cls)
    def wrap(*args,**kwargs):
        nonlocal instance
        if instance is None:
            instance = cls(*args,**kwargs)    #args,kwargs是用于将参数传递到__init__中
        return instance
    return wrap
@singleton
class A:
    pass
a = A()
a1 = A()
id(a)
id(a1)
```


##使用Supervisor管理后台服务遇到的问题

我想用Supervisor来监控后台的uwsgi服务，结果始终显示fatal报错，但是别的服务都可以，仔细研读文档才发现，原来Supervisor本身无法监控带有守护进程的服务，而wsgi本身的配置文件中有守护进程的属性，所以我在wsgi的配置文件中关闭了守护进程选项，直接依赖Supervisor的特性赋予uwsgi守护进程，如果uwsgi服务被kill则不需要依赖自身的守护进程拉起，而是变成依赖Supervisor拉起服务，这样就实现了Supervisor监控uwsgi服务

##使用Celery异步任务时遇到的问题

遇到了celery无法启动的问题，报错：SyntaxError: invalid syntax  ，这是因为我使用的python版本为最新3.7.3 ，而async已经作为关键字而存在了

在 celery 官方的提议下，建议将 async.py 文件的文件名改成 asynchronous。所以我们只需要将 celery\backends\async.py 改成 celery\backends\asynchronous.py，并且把 celery\backends\redis.py 中的所有 async 改成 asynchronous

另外虽然服务起来了，但是服务会不定期的假死

```
报错：Celery Process 'Worker' exited with 'exitcode 1' [duplicate]
```

经过搜索可以定位到问题所在，是因为celery依赖库billiard版本过低，导致任务发生了阻塞，所以最好的解决方案就是升级billiard

执行 pip install --upgrade billiard

官方的解释是，billiard最好>=3.5，所以如果不放心的话，还是指定版本号安装比较好

##使用轮询推送消息遇到的问题

有个需求，有新的产品上架时需要通过发消息提醒用户，但是做这个功能时，前端使用的是轮询的机制，每隔几秒往后台发请求，来确认是否有新产品上架，但是经过测试发现这种方式非常浪费http连接，同时效率很低，严重占用带宽资源，同时浏览器也变得非常卡，内存占用飙升。

对此，我将传统的http请求改成了websocket协议，由后端主动推送数据，前端并不需要判断逻辑，一次连接就可以完成功能，与此同时通过心跳包来保持长连接的稳定，完美的解决了这一问题。


##使用websocket遇到的问题（重点推荐）

在做消息主动推送功能时，由于用到了websocket长连接，当前端和后端都为一台服务器时，一对一对应，没有出现问题，但是当我为后台做负载均衡时，后台此时大于一台服务器，导致websocket无法对应上长连接的对象，导致服务出问题，经过排查，我将nginx负载均衡的策略由默认的轮询策略改成ip_hash的方式，这样session_id是唯一且一一对应的，解决了这个问题。


##前端使用富文本编辑器kindeditor跨域上传文件的问题

kindeditor在同域环境中是没有问题的，换句话说，也就是上传接口如果部署在前端页面同一个域名下是没有问题的，然而我的项目的系统架构是前后端分离，前端页面是vue.js服务，后端接口是django服务，分别部署在不同的服务器上，如果在vue.js页面中想要使用kindeditor中的上传文件功能，跨域请求django的接口就会报错。

解决思路就是用重定向方法来伪造成同域环境

我在前端新建一个redirect.html,用来伪造同域获取参数,后台django上传文件之后，不像之前那样返回json数据，而是带着参数直接重定向到做好的redirect.html页面，然后redirect.html页面利用js，再回到kindeditor页面，解决了跨域上传文件无法获取返回值的问题。

##mysql保留关键字问题

数据库查询时,由于表字段含有"order"/'comment'等和mysql自身的保留字名称一样;所以一查询就会报错

所以我写了一个装饰器方法，如果侦测到这些保留关键字，就加上反撇号来进行转义比如
```
select `order`,`comment`
```
这样就解决了这个问题

##mysql order by rand() 优化

当时有个需求是从数据库中随机取出一条数据作为展示，我理所当然的使用了 oder by rand(),结果导致慢查询非常多从而锁表影响了效率

查了一下文档

rand()会扫描整个表，然后再随机返回一个记录。对于比较小的表，通常不大于30万行记录的表，这种写法很实用。但是如果一旦记录大于了30万行，这个处理过程就会变得非常缓慢

所以结论就是能不用rand()就不要用

最后给出一种比较实用的替代方法的主要思想：

假设id是主键
首先：SELECT MIN(id), MAX(id) FROM tablename
然后：$id=rand($min,$max); //通过rand返回刚才取到的最大id和最小id之间的一个id号。
最后：SELECT * FROM tablename WHERE id='$id' LIMIT 1 

