## 关于不管你知不知道反正我现在才知道的各类Python小知识合集



#### 前言：

在工作中~~（或者无聊做的算法题中）~~有涉及到挺多的Python小知识、亦或者是新（sao）思（cao）路（zuo）；由于相信着自己的老年人记忆所以打算记录一下（以前写过的就不复读了）；
排序纯粹根据我遇到的顺序，所以有可能很浅显的在后面，要复习查看的时候记得Ctrl+F;

1、实例和对象的_\_dict\_\_是分开的，\_\_dir\_\_()才是包括继承来的；

```python
class A:
    def func(self):
        pass
    
a = A()
print(a.__dict__) # {}
print(a.__dir__()) # ['__module__', 'func', '__dict__', '__weakref__', '__doc__', '__repr__', '__hash__', '__str__', '__getattribute__', '__setattr__', '__delattr__', '__lt__', '__le__', '__eq__', '__ne__', '__gt__', '__ge__', '__init__', '__new__', '__reduce_ex__', '__reduce__', '__subclasshook__', '__init_subclass__', '__format__', '__sizeof__', '__dir__', '__class__']
a.b = 2
print(a.__dict__) # {'b': 2}
print(a.__dir__()) # ['b', '__module__', 'func', '__dict__', '__weakref__', '__doc__', '__repr__', '__hash__', '__str__', '__getattribute__', '__setattr__', '__delattr__', '__lt__', '__le__', '__eq__', '__ne__', '__gt__', '__ge__', '__init__', '__new__', '__reduce_ex__', '__reduce__', '__subclasshook__', '__init_subclass__', '__format__', '__sizeof__', '__dir__', '__class__']
```

可以看到执行了a.b赋值后下面的两个都有b了；另外要注意一下\_\_dir\_\_()是返回包括各种从祖宗类继承下来的属性列表，而\_\_dict\_\_返回的是自身特有的属性字典；也可以用Python自带的dir(a)来代替a.\_\_dir\_\_()；
另外值得一提的是Python对字典取值做个专门的优化，因此字典取值效率还是可以的（虽然我没严谨测试过），所以专门做出一个\_\_dict\_\_出来会不会是为了提高取各实例值的效率呢？

---

2、try组合里谁会执行的问题;

```python
try:
    print(t)
except:
    print(e)
else:
    print(e)
finally:
    print(f)
```

上面全是会报异常的代码，结果：

```python
NameError                                 Traceback (most recent call last)
<ipython-input-1-f2a0990762ee> in <module>
      1 try:
----> 2     print(t)
      3 except:

NameError: name 't' is not defined

During handling of the above exception, another exception occurred:

NameError                                 Traceback (most recent call last)
<ipython-input-1-f2a0990762ee> in <module>
      3 except:
----> 4     print(e)
      5 else:

NameError: name 'e' is not defined

During handling of the above exception, another exception occurred:

NameError                                 Traceback (most recent call last)
<ipython-input-1-f2a0990762ee> in <module>
      6     print(e)
      7 finally:
----> 8     print(f)

NameError: name 'f' is not defined
```

十分酷炫，报了3个异常，而且值得注意的是try里的异常也被报了出来；

改改：

```python
try:
    print(1)
except:
    print(2)
else:
    print(e)
finally:
    print(4)
```

这次只有else里的会报异常，结果：

```python
1
4
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-3-84448975d03c> in <module>
      4     print(2)
      5 else:
----> 6     print(e)
      7 finally:
      8     print(4)

NameError: name 'e' is not defined
```

finally里的代码会顶着异常执行，之后报出else里的异常；



然后就是如果里面是return的话会怎样，改下代码：

```python
def a():
    try:
        return 1
    except:
        return 2
    else:
        return 3
    finally:
        return 4
print(a()) # 4
```

大家都没事的话就会返回finally里的值（经测试try报错执行except的话最后还是finally）；然后：

```python
def a():
    try:
        return t
    except:
        return e
    else:
        return e
    finally:
        return 4
print(a()) # 4
```

之前的例子我们看到如果前面的有异常，finally会顶着异常执行完了之后再报异常，然而在方法里却没有报异常，而是直接获得返回值；然后：

```python
def a():
    try:
        return t
    except:
        return e
    else:
        return e
    finally:
        return f
print(a())
```

结果：

```python

---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-2-c762fada26a3> in a()
      2     try:
----> 3         return t
      4     except:

NameError: name 't' is not defined

During handling of the above exception, another exception occurred:

NameError                                 Traceback (most recent call last)
<ipython-input-2-c762fada26a3> in a()
      4     except:
----> 5         return e
      6     else:

NameError: name 'e' is not defined

During handling of the above exception, another exception occurred:

NameError                                 Traceback (most recent call last)
<ipython-input-2-c762fada26a3> in <module>
      8     finally:
      9         return f
---> 10 print(a())

<ipython-input-2-c762fada26a3> in a()
      7         return e
      8     finally:
----> 9         return f
     10 print(a())

NameError: name 'f' is not defined
```

如果finally里的也有异常，那就会把方法里的所有异常最后全都报出来；

所以在写finally时一定要小心，因为里面的代码会强制执行，不管前面有没有报错，尤其在类似方法里的return这样的地方，会让前面那一堆都显得没有意义；

---

3、使用dict的 **setdefault** 方便更新字典内对应的值

比较常见的一个需求就是统计一坨数据内各数据的出现频率，一般变量数据时会需要不断更新字典内某个特定键值的值；现在假如我要统计一段文字内各字母出现的频率，以前的话我是这样做的：

```python
data = 'fabwpeifjcqweuixcmqafajiVPChnqxgmoacfcjnaRHCw'
dic = {}
for char in data:
    dic.update({char: dic.get(char, 0) + 1})
```

虽然一行能解决，但是操作略骚且可读性比较烂，现在可以改成这样：

```python
data = 'fabwpeifjcqweuixcmqafajiVPChnqxgmoacfcjnaRHCw'
dic = {}
for char in data:
    dic.setdefault(char, 0)
    dic[char] += 1
```

效果一样的，会方（yōu）便（yǎ）不少，且用途不仅限于统计，很多需要字典经常更新对应值的情况都可以考虑使用；

抛砖引玉，上砖：

```python
data = [[123], '123', 123, {123}]
dic = {}
for d in data:
    dic.setdefault(d.__class__, []).append(d)
```

统计各类型的数据有哪些，或者说给数据按类型归类；

---

4、使用**collections.defaultdict**：

基本上玩过一阵子python的都知道用dict.get(key, default)来处理要获取字典里没有的key值，这比起处理KeyError要方便得多，但当要多次对字典进行取值处理时（不在循环内）每次都要手写一个default值似乎又有点麻烦，那这时候defaultdict就来了；

```python
from collections import defaultdict
dic = defaultdict(int)
dic[1]  # 0
```

注意defaultdict的第一个参数必须是callable的或者直接是个None，因此除非传None否则起码要个lambda；

defaultdict除了一开始定义那边有点区别以外，之后的用法就和普通的dict一样了，且之后的取值不需要每次都用get也不用except KeyError了，直接dict[key]即可

---

5、RSA加解密时的注意事项

使用RSA加解密时有些小坑，尤其是在客户端和服务器间使用rest传输时，现在先把简单的坑列举一下

**标明：采坑环境是：Python 3.7，Flask服务端，rsa库，requests库**

​	1）一定要注意公钥和私钥的格式是否符合库的要求，直接从网上复制下来的密钥对不一定符合你使用的RSA库的标准，最好使用库本身自带的生成方式；

​	2）使用RSA库的load_pkcs1方法时注意你导入的字符串要先encode一下，这个方法只接收bytes类型而不接受string；

​	3）注意请求的封装格式（跟RSA无关，但是也要注意，尤其像我这种菜鸟）；

​	4）由于rest方法传输的数据虽说是json但本质还是string，而使用rsa.encrypt方法生成的密文是\x开头的16进制数据，直接转成string会产生乱码，因此在post出去之前要先使用base64进行编码，就像这样：

```python
import base64
import rsa
import json
import requests

data = {'aaa': 111}
public_key = 'your public key'
enc_data = rsa.encrypt(json.dumps(data).encode(), rsa.PublicKey.load_pkcs1(public_key.encode()))
requests.post('http://www.baidu.com', data={'data': base64.encodebytes(enc_data)})
```

不然传过去服务端收到的是乱码，最终解码失败；然后服务器端也要处理一下(用的是flask，其他的类似这样改就行)：

```python
import base64
import rsa
import json
import requests

private_key = 'your private key'
dec_data = rsa.decrypt(base64.decodebytes(request.values.get('data').encode()), rsa.PrivateKey.load_pkcs1(private_key.encode()))
```

需要再用一次base64.decodebytes来解码回原来的样子（bytes）再通过RSA解密，这样才能得到正确的明文；

​	5）1024位的密钥最多只能加密117位（密钥长度 / 8 - 11，所以2048位的密钥可以加密的明文长度为 2048 / 8 - 11 = 245位）的明文，所以加密时一定要注意，如果过长就要按照每117位来分组加密，例如（不重复导入了）：

```python
import math

response_message = json.dumps({'aaa': 222})
res_bytes = response_message.encode()
max_len = 117
res_datas = []
for i in range(math.ceil(len(res_bytes) / max_len)): # 记得要上取整
    res_data = rsa.encrypt(
        res_bytes[i * max_len: (i + 1) * max_len], 
        rsa.PublicKey.load_pkcs1(public_key.encode())
    )
    res_datas.append(res_data)
return b''.join(res_datas)
```

---

6、bool是int的子类

是的你没看错，平时写的 True == 1是成立的这点虽然是常识，但是原因是因为bool是int的子类这件事可能就不一定谁都知道了，日常判断时要小心坑，比如：

```python
isinstance(True, int)  # True
```

这是成立的，这TM居然是成立的；

证明：

```python
print(True.__class__.__base__)  # int
```

