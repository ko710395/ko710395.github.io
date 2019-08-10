## *args 和 **kwargs

一对魔法参数（因为能实现很神奇的功能所以叫魔法，虽然弄懂原理之后还是走近科学），当函数的参数无法确定的时候会使用；



这两个参数其实只是约定俗成的命名，并不一定要用args或kwargs，只要有*和**即可，你甚至可以用\*cxk和\*\*ctrl（是ctrl不是唱跳rap篮球）来命名;



前者是将参数打包成tuple传入函数，比如这样：

```python
def p(*args):
        print(args)
p(1,2,3)
```

结果：

```
(1, 2, 3)
```

这样无论函数有多少个参数，都能通过这个打包成元组传入函数，从而实现不定参数的函数（print函数就是如此，无论输入多少个参数都能打印出来）；



后者则是将参数的作为键值对打包成dict传入函数，比如这样：

```python
def p(**kwargs):
        print(kwargs)
p(a=1,b=2,c=3)
```

结果：

```
{'a': 1, 'b': 2, 'c': 3}
```

这样就能实现一些带上指定参数的函数，又比如print函数

```python
print(*objects, sep=' ', end='\n', file=sys.stdout)
```

虽然我们平时用print很方便，但是实际上它也有一些参数，如sep表示print每一项中间用什么隔开，end表示用什么结尾等；这些配置类型的参数就需要打包成dict传入函数；



值得一提的是，前者*args在任何参数的任何地方都能使用，就像：

```python
def p(a,*args,b):
  pass
def q(*args,a,b):
  pass
```

都没有问题，但是**kwargs就只能放在最后使用：

```python
def p(a,*args,b,**kwargs):
  pass
```

~~别问我为什么~~

