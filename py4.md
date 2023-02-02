Help()函数是一个内置函数，用于查看函数或模块用途的详细说明：

```
import copy
help(copy.copy)
```

Dir()函数也是Python内置函数，dir() 函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表。

```
dir(copy.copy)
```

这些都可以归结为自省，主要就是在遗忘内置方法或者属性时，快速回想起来


## 反射

反射就是通过字符串的形式，导入模块；通过字符串的形式，去模块寻找指定函数，并执行。利用字符串的形式去对象（模块）中操作（查找/获取/删除/添加）成员，一种基于字符串的事件驱动！

```
1. getattr()函数是Python自省的核心函数，具体使用大体如下：
class A: 
def __init__(self): 
self.name = 'zhangjing'
#self.age='24'
def method(self): 
print"method print"
  
Instance = A() 
print getattr(Instance , 'name, 'not find') #如果Instance 对象中有属性name则打印self.name的值，否则打印'not find'
print getattr(Instance , 'age', 'not find') #如果Instance 对象中有属性age则打印self.age的值，否则打印'not find'
print getattr(a, 'method', 'default') #如果有方法method，否则打印其地址，否则打印default 
print getattr(a, 'method', 'default')() #如果有方法method，运行函数并打印None否则打印default 

2. hasattr(object, name)

说明：判断对象object是否包含名为name的特性（hasattr是通过调用getattr(ojbect, name)是否抛出异常来实现的）

3. setattr(object, name, value)

这是相对应的getattr()。参数是一个对象,一个字符串和一个任意值。字符串可能会列出一个现有的属性或一个新的属性。这个函数将值赋给属性的。该对象允许它提供。例如,setattr(x,“foobar”,123)相当于x.foobar = 123。

4. delattr(object, name)

与setattr()相关的一组函数。参数是由一个对象(记住python中一切皆是对象)和一个字符串组成的。string参数必须是对象属性名
```