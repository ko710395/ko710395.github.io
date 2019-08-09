# 装饰器

### **目的**：

是在不修改原函数(类)的代码以及调用方式的前提下为原函数(类)增加额外功能，如变量、约束条件等；

### **原理**：

利用闭包的原理，设置两层函数，第一层函数以一个函数为参数，return第二层函数；第二层参数与需要修饰的函数的参数格式相同(一般采用**args)，return的值根据实际使用情况来设置；

### **例子**：

```python
# 装饰器函数
def decorator(fuc):
    def inner():
        print("登录完成")
        return fuc()
    return inner

# 使用装饰器语法糖
@decorator
def comment():
    print("这里发表评论")
# 注释掉原来该处代码  
# comment = decorator(comment)
comment()
```

### 结果：

```
登录完成
这里发表评论
```



# 闭包

### 目的：

函数执行完毕后仍能保持当前的运行环境(比如保存非全局的外部局部变量)；引申一下还能用于根据参数不同来实现不同功能的函数（输入a做加法输入b做减法之类的）；

### 定义：

1、函数下再定义函数；

2、内层函数调用了外层函数的参数；

3、外层函数返回了内层函数；

(其实就是装饰器的必要不充分形式，**外层函数的参数也是个函数**且**内层函数的参数形式和作为外层函数的参数的函数参数格式一致**时，该闭包即为一个装饰器)

### 原理：(个人理解)

有点类似class那样，变量全都限制在外层函数内，这些变量参数对于内层函数来说就是外部变量，但是对于全局来说这些变量却是外层函数的局部变量，这样就能在不影响全局变量的前提下让内层函数保留外部变量参数的前提下多次执行；

​		~~有点像多划分一层沙盒来隔离全局污染？~~



# map和reduce函数

### map函数：

```python
def plus(a):
    return a+1

li = [1,2,3,4,5]
a = map(plus,li)
print(list(a))

```

结果：[2, 3, 4, 5, 6]

map函数接收两个参数：一个函数（ps：python中的数据类型强制转换也是一个函数）和一个list，map会将list中的每一个元素都分别作为参数传入函数中执行，执行结果返回为一个map对象，如上述例子所示，需要转换成list打印，否则会返回内存地址；

### reduce函数：

```python
from functools import reduce
def plus(a,b):
    return a+b
li = [1,2,3,4,5]
a = reduce(plus,li)
print(a)
```

结果：15

其实就是map的2个参数版，等价于plus(plus(plus(plus(1,2),3),4),5)，固定为2个参数；与map函数不同的一点是reduce能根据返回的结果类型直接调用；

**使用reduce前要先 from functools import reduce**

以上两个函数都只是相对复杂的计算抽象化，完全可以用基础函数代替，但是有现成的为啥不用呢？ ~~当然要优雅！~~



### 匿名函数lambda

```python
lambda x: x * x
```

实际上就等价于

```python
def f(x):
    return x * x
```

lambda函数冒号后面的表达式就是其返回值，冒号前的x表示参数，lambda可以 **有效防止函数重命名的问题**，以及在需要简单函数定义的时候**有效简化代码**；

甚至还可以用于闭包return：

```python
def build(x, y):
    return lambda: x * x + y * y
```



## 偏函数：

目的：**固定输入某个函数的某个参数，方便复用**；

例子：当需要int强制转换时需要都转成二进制时，不需要每次都输入参数base=2

```python
int(100,base=2)
```

而使用了偏函数后，可以定义一个int2函数来替代int(，base=2)，不需要多次输入base=2：

```python
import functools
int2 = functools.partial(int,base=2)
int2(100)
```

这样就能省点事了。 ~~似乎用处不大？~~



## With语句：

参考：https://www.cnblogs.com/DswCnblog/p/6126588.html

先上例子：

```python
class Sample:
    def __enter__(self):
        print "In __enter__()"
        return "Foo"
 
    def __exit__(self, type, value, trace):
        print "In __exit__()"
 
def get_sample():
    return Sample()
 
with get_sample() as sample:
    print "sample:", sample
```

结果：

```python
In __enter__()
sample: Foo
In __exit__()
```

先根据结果来看看发生了啥：

- \__enter__()执行了；
- \__enter__()返回的值，即"Foo"，赋值给了"sample"，并执行了下面那行的代码块；
- \__exit__()执行了；



因此，可以将with语句的行为分解成几个步骤：

- 执行with后面跟着的对象(或者返回的对象)的\__enter__()方法；
- 将刚执行的\__enter__()返回的值赋值给as后面跟着的变量；←如果有as语句的话，没有则略过这一步；
- 执行冒号下面的代码段；
- 最后执行with后面跟着的对象(或返回的对象)的\__exit__()方法，即使上一步的代码段中出现异常；



所以，从步骤中可以看出，with语句执行时的**条件**，那就是：

​					**with后跟着的对象要有\__enter__()和\__exit__()方法**

当然这个对象可以是自己，即enter里return self；



那么，with语句有什么用呢？什么时候能用到呢？



平时用的比较多的应该就是任何需要收尾的工作执行时，比如IO的open和close；事实上，python内置的open函数所在的类已经写好了\__enter__()和\__exit__()方法，学的时候经常是这样的：

```python
f = open('/path/to/file', 'r')
print(f.read())
f.close()
```

打开文件→执行你需要的代码→关闭文件；

但python很懂，它内置的类里就有\__enter__()和\__exit__()方法，因此我们可以这样使用：

```python
with open('/path/to/file', 'r') as f:
    print(f.read())
```

这样我们就不需要再调用close()，会自动调用close()以防止我们写完了长长的代码块后长舒一口气结果忘了close()导致出问题；所以一般来说，exit方法内会写代码的收尾工作，比如关闭文件，释放资源等，这在编写库时非常常用，而常用的库里一般也写好了exit方法供开发者用with调用；



当然，with的语句可不止这点用途，它更大的用途是：**处理异常**



再来一个例子：

```python
class Sample:
    def __enter__(self):
        return self
 
    def __exit__(self, type, value, trace):
        print "type:", type
        print "value:", value
        print "trace:", trace
 
    def do_something(self):
        bar = 1/0
        return bar + 10
 
with Sample() as sample:
    sample.do_something()
```

结果：

```python
type: <class 'ZeroDivisionError'>
value: division by zero
trace: <traceback object at 0x10f7c8ec8>
Traceback (most recent call last):
  File "space.py", line 19, in <module>
    sample.do_something()
  File "space.py", line 15, in do_something
    bar = 1/0
ZeroDivisionError: division by zero
```

终端会返回这些信息，可以看出，代码块部分将异常抛给\__exit__()进行处理，因为exit中没有对异常进行捕获，所以最终抛给了终端打印了出来，但是可以发现，**无论代码块中是否出现异常，exit方法都会执行**，利用这一点我们就能方便地处理代码块执行过程中可能出现的异常，比如处理用户输入时。



但是这样看exit方法似乎有些魔幻，那么接下来详细介绍一下exit方法：



**参考：https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/**



那么exit的工作原理是怎样的呢？上面链接文章的原文是：

**如果执行过程中没有出现异常，或者语句体中执行了语句 break/continue/return，则以 None 作为参数调用\__exit\__(None, None, None) ；如果执行过程中出现异常，则使用 sys.exc_info 得到的异常信息为参数调用 \__exit\__(exc_type, exc_value, exc_traceback）**

**出现异常时，如果 \__exit\__(type, value, traceback) 返回 False，则会重新抛出异常，让with 之外的语句逻辑来处理异常，这也是通用做法；如果返回 True，则忽略异常，不再对异常进行处理。**

当你需要自定义exit方法时，就能根据上面的定义自行编写exit方法；

值得一提的是，当代码块中出现异常时，会中断代码块里的代码，转而执行exit方法，这点需要注意；



## *args 和 **kwargs

一对魔法参数，当函数的参数无法确定的时候会使用；



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

这样无论函数有多少个参数，都能通过这个打包成元组传入函数，从而实现不定参数的函数；



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



## 深拷贝与浅拷贝

首先，python中的变量能分为**不可变数据**和**可变数据**两大类，前者包括Number数字、String字符串和Tuple元组，后者包括List列表、Dictionary字典和Set集合；



python自带了一个copy包，使用时需要先import；浅拷贝则使用copy包里的copy方法，深拷贝则使用里面的deepcopy方法；



#### 浅拷贝：

对于不可变类型（Number、String、Tuple）数据，浅拷贝只复制地址指向，即拷贝后两个变量指向的内存地址相同，并不会开辟新的内存；

对于可变类型（List、Dictionary、Set）数据，浅拷贝会开辟新的内存，使用后内存内将会有两端内存放着相同的数据；



#### 深拷贝：

同浅拷贝，只是会进一步递归拷贝所有子元素，即将所有子元素都进行拷贝；



测试如下：

```python
import copy
a = 1
b = 2
li1 = [a,b,[3,4]]
li2 = copy.copy(li1)
li3 = copy.deepcopy(li1)
print(id(li1),id(li2),id(li3))
print(id(li1[0]),id(li2[0]),id(li3[0]))
print(id(li1[2]),id(li2[2]),id(li3[2]))
```

结果：

```
4379996232 4379974920 4379971976
4376334368 4376334368 4376334368
4379764040 4379764040 4380000968
```

可以看到，对一个list进行浅拷贝后，新的list本身的内存地址不同于原来的list，符合浅拷贝的操作流程，但是其子元素里也有个list，而对于list的浅拷贝应该是开辟新的内存地址，而li2中的子list和li1中的子list的内存地址一样，所以浅拷贝只是拷贝了一个list，子元素内的内存地址都是照搬的；而深拷贝得到的li3的子list的内存却不同于li1，因此可以得出结论：

**浅拷贝时子元素内存地址是照搬的，而深拷贝时则是递归浅拷贝子元素**

所以会产生出的区别就是：

**进行浅拷贝时若子元素中有可变类型数据，则当对该数据进行修改时原数据也会跟着改，而如果是进行深拷贝则不会修改**

再举个例子：

```python
import copy
li = [1,2,[3,4]]
li2 = copy.deepcopy(li)
li3 = copy.copy(li)
print(li,li2,li3)
li2[2].append(8)
li3[2].append(9)
print(li,li2,li3)
```

结果：

```python
[1, 2, [3, 4]] [1, 2, [3, 4]] [1, 2, [3, 4]]
[1, 2, [3, 4, 9]] [1, 2, [3, 4, 8]] [1, 2, [3, 4, 9]]
```

可以看到，li2是li的深拷贝，li3是li的浅拷贝，分别对li2和li3中的可变数据进行修改时原数据li是跟着浅拷贝对象li3改变的；

