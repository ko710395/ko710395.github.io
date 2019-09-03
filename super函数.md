​		看坑爹odoo的时候看到了有个不知道是啥的super函数，一查发现不是odoo的是python的，然后看了一下介绍感觉没懂，过了一段时间后又研究了一下，又有点似懂非懂，总之先记录下来我的理解，希望是对的；

​		根据介绍，super函数是用来解决类继承的问题；众所周知，python中声明了一个类的对象后就会自动跑构造函数\__init\_\_()，一般构造函数里会放一些类的变量，比如这样：

```python
class book():
    def __init__(self):
        self.name = "book"
        print("I'm a book!")
    def pr(self):
        print(self.name)
```

然后我们再写一个类，继承自这个类：

```python
class novel(book):
    def __init__(self):
        self.novel=True
        print("I'm novel!")
    def print_name(self):
        print(self.novel)
```

然后我们运行：

```python
n = novel() # I'm novel!
n.print_name() # True
n.pr() #  AttributeError: 'novel' object has no attribute 'name'
```

是的报错了，因为构造函数中定义的变量无法继承，当然此时就会想，那我的变量放在构造函数外声明不就好了？确实是，但如果有这种需求呢？

```python
class book():
    def __init__(self,name):
        self.name = name
        print("I'm a book!")
    def pr(self):
        print(self.name)
```

类似这样，需要根据参数来给变量赋值的时候，当然可能还是有别的实现方法的，不一定非要这样，但是我们想这样写，因为这样才符合风格，但是我们又想让子类继承这个变量，那该怎么办呢？

方法还是有的：

```python
class novel(book):
    def __init__(self):
        self.novel=True
        book.__init__(self)
        print("I'm novel!")
    def print_name(self):
        print(self.novel)
```

直接这样来显性调用一次就好了；

但是这样的话，假如这个类被继承了很多次，然后后续我们又修改了这个类，岂不是他的所有子类都要跟着改？

然后super函数来了：

```python
class novel(book):
    def __init__(self):
        super().__init__()
        self.novel=True
        print("I'm novel!")
    def print_name(self):
        print(self.novel)
```

ps：python3中可以直接super().xxx()来调用而不需要super(class,self).xxx()

这样super函数就会直接寻找当前类的父类去执行相应方法，而不需要每个类名都去写；

以上是单继承的情况，其实严格来说问题也不大，毕竟一般来说写好的父类一般不会去大幅度修改；

那么super函数真正发挥威力的时候（？），就是有多重继承的时候了：

```python
class A():
    def __init__(self):
        print("This is A's init!")


class B(A):
    def __init__(self):
        A.__init__(self)
        print("This is B's init!")


class C(A):
    def __init__(self):
        A.__init__(self)
        print("This is C's init!")


class D(B, C):
    def __init__(self):
        B.__init__(self)
        C.__init__(self)
        print("This is D's init!")

    def pr(self):
        print("This is D's pr!")


d = D()
```

结果：

```
This is A's init!
This is B's init!
This is A's init!
This is C's init!
This is D's init!
```

这种继承结构时，如果我们需要父类构造函数的时候一般会这样做，但是这样的话可以看到，不可避免地class A的构造函数被执行了2次；

然后我们改用super函数：

```python
class A():
    def __init__(self):
        print("This is A's init!")


class B(A):
    def __init__(self):
        super().__init__()
        print("This is B's init!")


class C(A):
    def __init__(self):
        super().__init__()
        print("This is C's init!")


class D(B, C):
    def __init__(self):
        super().__init__()
        print("This is D's init!")

    def pr(self):
        print("This is D's pr!")


d = D()
```

结果：

```
This is A's init!
This is C's init!
This is B's init!
This is D's init!
```

这样就能有效避免父类的构造函数被多次调用；

实际应用场景我暂时也想不到，反正有这么个手段在这儿，当以后遇到父类构造函数被强制多次调用时可以想起有这么个super函数就行了；





[返回首页](https://ko710395.github.io/)