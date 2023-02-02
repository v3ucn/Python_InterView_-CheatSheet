在程序设计中，鸭子类型（英语：duck typing）是动态类型的一种风格。 在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由当前方法和属性的集合决定。

# 体现多态性

```
from abc import ABCMeta,abstractmethod #(抽象方法)

class Payment(metaclass=ABCMeta):   # metaclass 元类  metaclass = ABCMeta表示Payment类是一个规范类
    def __init__(self,name,money):
        self.money=money
        self.name=name

    @abstractmethod      # @abstractmethod表示下面一行中的pay方法是一个必须在子类中实现的方法
    def pay(self,*args,**kwargs):
        pass

class AliPay(Payment):

    def pay(self):
        # 支付宝提供了一个网络上的联系渠道
        print('%s通过支付宝消费了%s元'%(self.name,self.money))

class WeChatPay(Payment):

    def pay(self):
        # 微信提供了一个网络上的联系渠道
        print('%s通过微信消费了%s元'%(self.name,self.money))


class Order(object):

    def account(self,pay_obj):
        pay_obj.pay()

pay_obj = WeChatPay("wang",100)
pay_obj2 = AliPay("zhang",200)

order = Order()
order.account(pay_obj)
order.account(pay_obj2)

```

## 鸭子类型

```
#示例1
class Animal():
    def who(self):
        print("I am an Animal")
class Duck(Animal):
    def who(self):
        print("I am a duck")

class Dog(Animal):
    def who(self):
        print("I am a dog")

class Cat(Animal):
    def who(self):
        print("I am a cat")

def func(obj):
    obj.who()

if __name__ == "__main__":
    duck=Duck()
    dog=Dog()
    cat=Cat()
    func(duck)
    func(dog)
    func(cat)
```

示例2中定义一个函数func()，这个函数对参数有一个要求，那就是参数必须有who()这个方法。不管你是什么对象，是duck对象也好，是dog对象也罢，只管对象实现who()方法就可。这就是鸭子类型，它根本不管你是什么对象，只要你有这个方法，有这个行为，表现得像鸭子，走起来像鸭子，游泳起来鸭子，叫起来也像鸭子，那么尽管你是一只会飞天的猪，也是称为鸭子类型。