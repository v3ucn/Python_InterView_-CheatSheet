#解释一下Python中的三元运算

三元运算其实就是横过来的 if else 表达式

```
[on true] if [expression] else [on false]
```
如果表达式为True，就执行[on true]中的语句。否则，就执行[on false]中的语句

```
a,b=2,3
min=a if a<b else b
min
```

一般情况下，三元运算符就是在赋值变量的时候，可以直接加判断，然后赋值，让判断和复制变成连贯操作