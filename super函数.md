​		重新写这篇笔记，最近有时间重新研究了一波super函数，理解比上次透彻了，于是再写一次；上次的内容由于理解不够透彻，很多都是介绍表象的，所以这次就在之前那篇的基础上进行补充说明；

### 啥时用super？

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

### 多重继承时

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

### super的工作过程

由上面的试验可以看出，super似乎就是指代当前类的父类（直到object类）；假设真是如此，那上面多重继承的例子里就不应该会出现区别，因为B的父类是A，如果super()就是父类的话那在B中使用A()和super()._\_init__()就应该没有区别，但实际上呢？

我们把代码改成这样：

```python
class A():
    def __init__(self):
        print("This is A's init!")


class B(A):
    def __init__(self):
        A()
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
This is B's init!
This is D's init!
```

我的C呢？我那么大个C呢？

所以，结论就是：

##### super函数并不完全指代父类

那就有必要弄清楚super的原理了，不然使用不理解的东西就是隐患；

查了一大波文章资料后，得知：**super函数的执行顺序是根据调用对象所在的累的mro表决定的**；

mro(Method Resolution Order)表，在python中是用来决定方法解析顺序的；在python中类分为经典类（classic class）和新式类（new-style class），后者是python2.2后引进的，而在python3后默认都是新式类，3之前2.2之后如果不是显式继承object类的都是经典类；两者最大的区别就是这个mro表，经典类是采用深度优先，而新式类则是广度优先，在此就不在多讨论两种优先的区别了反正我用的python3都是新式类；

说回这个mro表，在python3中可以使用cls.mro()查看这个类的mro表，比如上面那个例子：

```python
print(D.mro())  # <class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
```

这个表具体是怎么得到的，我之后再[单独写一篇](https://ko710395.github.io/C3算法)来记录一下；总之每个类都有这么个表，像记录了自己的族谱一样记录了从这个类本身开始一直到object类中继承到的类；

super本身是一个类，它可以传入参数，像这样：

```python
super(cls, obj)
```

前面的是一个类，后面是一个对象，一般是一个类的实例；当你想我那样在类方法里面调用时就可以写成不带参数的样子，默认是传入当前的类和self，因此super()这种写法仅限于类方法里面使用；

##### 其实super函数的顺序就是跟着后面的obj所在的类的mro表（obj.\__class__.mro()）来执行的；

回到ABCD都用super的那个例子；根据结果来看，四个函数的执行顺序是ACBD，也就是说在D中执行的super().\__init__()分别执行了ACB，我们改写这个例子：

```python
class A():
    def __init__(self):
        print('Enter A!')
        print('Leave A!')

class B(A):
    def __init__(self):
        print('Enter B!')
        super().__init__()
        print('Leave B!')


class C(A):
    def __init__(self):
        print('Enter C!')
        super().__init__()
        print('Leave C!')


class D(B, C):
    def __init__(self):
        print('Enter D!')
        super().__init__()
        print('Leave D!')



d = D()
print("D's mro is", D.mro())
print("B's mro is", B.mro())
```

结果：

```
Enter D!
Enter B!
Enter C!
Enter A!
Leave A!
Leave C!
Leave B!
Leave D!
D's mro is [<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
B's mro is [<class '__main__.B'>, <class '__main__.A'>, <class 'object'>]
```

可以看到，执行的顺序是DBCA，这和D类的mro表顺序一致；而B的mro表则没有C；

但是为什么执行到B的时候函数会知道要跟着D的mro表而不是自己的mro表来继续执行呢？这个表是从哪里传进B里的呢？这就是我想了很久才理解的问题（人蠢没药医）；

##### 奥秘其实就在函数里的self和super函数的默认参数；

我们再改写一下例子的函数：

```python
class A():
    def __init__(self):
        print('Enter A!')
        print("A's self is", self)
        print('Leave A!')

class B(A):
    def __init__(self):
        print('Enter B!')
        print("B's self is", self)
        super().__init__()
        print('Leave B!')


class C(A):
    def __init__(self):
        print('Enter C!')
        print("C's self is", self)
        super().__init__()
        print('Leave C!')


class D(B, C):
    def __init__(self):
        print('Enter D!')
        super().__init__()
        print('Leave D!')


d = D()
print(D.mro())
```

结果：

```
Enter D!
Enter B!
B's self is <__main__.D object at 0x10e153f90>
Enter C!
C's self is <__main__.D object at 0x10e153f90>
Enter A!
A's self is <__main__.D object at 0x10e153f90>
Leave A!
Leave C!
Leave B!
Leave D!
[<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
```

从D调用super时，执行到他的所有父类的时候，传进去的self都是同一个，就是D的实例d；

```python
class A:
    def a(self):
        print("I'm A", self)

class B(A):
    pass

b = B()
b.a()  # I'm A <__main__.B object at 0x10792c210>
```

用子类的对象去调用父类的方法时，使用的self就是子类的实例，如果一路用super往上调用的话这个实例就会一直作为调用的实例（就是执行时的self）往上传；因此传这个实例就是将这个mro表往上传的方法；

在上面这个例子的情况下，执行到B的时候super的参数是super(B, d)，因此super会读取d所在类（d.\__class__，即D）的mro表，从前一个参数，即B开始往后查，在D的mro表中，B的下一个是C，因此super下一步会执行C的方法，所以此时**super函数指代的并不是父类，而是B的兄弟类C**；

##### 综上所述，其实super函数指代的是它第一个参数、即那个类（在类方法中的默认值是当前类），在它第二个参数、即那个实例所在的类的mro表中的下一个类，而不是简单粗暴的父类；

当第一个参数的类不存在于第二个参数的实例所在类的mro表中时则认为不合法，super会报错；



弄懂了之后就神清气爽了！





[返回首页](https://ko710395.github.io/)