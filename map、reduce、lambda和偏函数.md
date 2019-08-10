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

这样就能省点事了。

感觉用处不大？来个实际应用的例子：

```python
import functools
print2 = functools.partial(print,end = ',之后是')
li = [1,2,3,4,5]
for i in li[:-1]:
    print2(i)
print(li[-1])
```

输出结果：

```
1,之后是2,之后是3,之后是4,之后是5
```