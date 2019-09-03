## With语句：

参考：https://www.cnblogs.com/DswCnblog/p/6126588.html ~~其实就是转述~~

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

 



[返回目录](https://ko710395.github.io/)