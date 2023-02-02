猴子补丁（monkey patch）的主要功能就是动态的属性的替换。虽然属性的运行时替换和猴子也没什么关系，所以说猴子补丁的叫法有些莫名其妙，但是只要和“模块运行时替换的功能”对应就行了。

monkey patch允许在运行期间动态修改一个类或模块（注意python中一切皆对象，包括类、方法、甚至是模块）

```
class A:
    def func(self):
        print('Hi')
    def monkey(self):
        print('Hi, monkey')
a = A()
A.func=A.monkey   #在运行的时候，才改变了func
a.func()
'''运行结果
Hi, monkey
'''
```

# 使用场景

这里有一个比较实用的例子，很多代码用到 import json，后来发现ujson性能更高，如果觉得把每个文件的import json 改成 import ujson as json成本较高，或者说想测试一下用ujson替换json是否符合预期，只需要在入口加上：

```
import json  
import ujson  

def monkey_patch_json():  
    json.__name__ = 'ujson'  
    json.dumps = ujson.dumps  
    json.loads = ujson.loads  

monkey_patch_json()
```