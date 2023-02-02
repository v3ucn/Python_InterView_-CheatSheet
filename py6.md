当我们不知道向函数传递多少参数时，比如我们向传递一个列表或元组，我们就使用*args。

```
>>> def func(*args):
    for i in args:
        print(i)  
>>> func(3,2,1,4,7)
```

在我们不知道该传递多少关键字参数时，使用**kwargs来收集关键字参数。

```
>>> def func(**kwargs):
    for i in kwargs:
        print(i,kwargs[i])
>>> func(a=1,b=2,c=7)
```

