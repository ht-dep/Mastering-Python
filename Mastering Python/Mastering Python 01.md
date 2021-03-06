
# 1.准备
> 1.说明：《Mastering Python》读书笔记

> 2.要求：首先，希望你是处在`python3`版本，然后拥有一个干净的虚拟环境更是必要。

> 3.virtualenv或者anaconda都是不错的选择

# 2.Pythonic Syntax, Common Pitfalls, and Style Guide
## 2.1.Pythonic code
对于python开发者，无不希望写出pythonic风格的代码，请谨记以下几点：
    - Clean
    - Simple
    - Beautiful
    - Explicit
    - Readable
具体的话请打开你的python解释器：
```
import this
```
细细感受吧。
### 2.1.2.Differences between value and identity comparisons
对于下面这个例子：


```python
a1 = 200 + 56
a2 = 256
b1 = -4 - 1
b2 = -5
c1 = 200 + 57
c2 = 257
d1 = -4 - 2
d2 = -6
print('%r == %r: %r' % (a1, a2, a1 == a2))
print('%r is %r: %r' % (a1, a2, a1 is a2))
print('%r == %r: %r' % (b1, b2, b1 == b2))
print('%r is %r: %r' % (b1, b2, b1 is b2))
print('%r == %r: %r' % (c1, c2, c1 == c2))
print('%r is %r: %r' % (c1, c2, c1 is c2))
print('%r == %r: %r' % (d1, d2, d1 == d2))
print('%r is %r: %r' % (d1, d2, d1 is d2))
```

    256 == 256: True
    256 is 256: True
    -5 == -5: True
    -5 is -5: True
    257 == 257: True
    257 is 257: False
    -6 == -6: True
    -6 is -6: False


可以看到，在值相同的情况下，使用`==`比较都返回`True`自不必说，为何有的`id`相同有的`id`却不同？

这是因为，在`pyhton`中，会提前为一小部分整数提前分配空间，这个整数的范围是\[-5,256\]，当整数范围大于256或者小于-5时，`python`便会为该整数创建对象，于是造成id不同。如果想深入了解，可阅读[深入 Python 整数对象的实现](http://python.jobbole.com/82632/)
### 2.1.3.Loops
对于一个列表，比如：
``` python
my_list = [1, 2, 3, 4]
```
想将其输出为`[index:value]`这种列表格式，请看以下几种实现方法：
``` python
index = 0
my_list = [1, 2, 3, 4]
new_list = []
for value in my_list:
    value = "{0}:{1}".format(index, value)
    new_list.append(value)
    index += 1
print(new_list)
```
不得不说，实在很繁琐，我们可以利用`python`提供的`enumerate`类：
``` python
new_list = []
for index, value in enumerate(my_list):
    value = "{0}:{1}".format(index, value)
    new_list.append(value)
print(new_list)
```
第二种方法确实精简了不少，但是，优雅的`pythonic`不是浪得虚名，请看这种：


```python
my_list = [1, 2, 3, 4]
["{0}:{1}".format(index, value) for index, value in enumerate(my_list)]
```




    ['0:1', '1:2', '2:3', '3:4']



## 2.2.Common pitfalls
虽说`python`自诩是一种清晰易读无歧义的优雅语言，但是这个伟大的目标并不是那么容易实现。
### 2.2.1.Function arguments
需要知道的是，**千万不要使用可变参数作为函数默认参数**，看下面这样一个例子：


```python
def spam(key, value, list_=[], dict_={}):
    list_.append(value)
    dict_[key] = value
    print('List: %r' % list_)
    print('Dict: %r' % dict_)


spam('key 1', 'value 1')
spam('key 2', 'value 2')
```

    List: ['value 1']
    Dict: {'key 1': 'value 1'}
    List: ['value 1', 'value 2']
    Dict: {'key 2': 'value 2', 'key 1': 'value 1'}


按照代码的思路，输出的结果应该是下面这样才对：
``` python
List: ['value 1']
Dict: {'key 1': 'value 1'}
List: ['value 2']
Dict: {'key 2': 'value 2'}
```
实际上并非如此，首先，`list`以及`dict`都是可变对象，当`spam`函数被定义，`list_`和`dict_`就会被创建（**但是它们只被创建一次**），随后调用函数的时候，`list_`和`dict_`并没有被重新创建，于是，这个陷阱便出现了。

正确的做法：


```python
def spam(key, value, list_=None, dict_=None):
    if list_ is None:
        list_ = []
    if dict_ is None:
        dict_ = {}
    list_.append(value)
    dict_[key] = value
    print('List: %r' % list_)
    print('Dict: %r' % dict_)


spam('key 1', 'value 1')
spam('key 2', 'value 2')
```

    List: ['value 1']
    Dict: {'key 1': 'value 1'}
    List: ['value 2']
    Dict: {'key 2': 'value 2'}


### 2.2.2.Class properties
在类的定义中，也存在这样的问题。
``` python
class Spam(object):

    list_ = []
    dict_ = {}

    def __init__(self, key, value):
        self.list_.append(value)
        self.dict_[key] = value
        print('List: %r' % self.list_)
        print('Dict: %r' % self.dict_)


Spam('key 1', 'value 1')
Spam('key 2', 'value 2')
```
output:
```
List: ['value 1']
Dict: {'key 1': 'value 1'}
List: ['value 1', 'value 2']
Dict: {'key 1': 'value 1', 'key 2': 'value 2'}
```
替代方案：


```python
class Spam(object):
    def __init__(self, key, value):
        self.list_ = [key]
        self.dict_ = {key: value}
        print('List: %r' % self.list_)
        print('Dict: %r' % self.dict_)


Spam('key 1', 'value 1')
Spam('key 2', 'value 2')
```

    List: ['key 1']
    Dict: {'key 1': 'value 1'}
    List: ['key 2']
    Dict: {'key 2': 'value 2'}





    <__main__.Spam at 0x10498a2b0>



### 2.2.3.Modifying variables in the global scope
在函数内对变量进行操作也有需要注意的地方。


```python
spam = 1


def eggs():
    spam += 1
    print(spam)


eggs()
```


    ---------------------------------------------------------------------------

    UnboundLocalError                         Traceback (most recent call last)

    <ipython-input-8-52c0d52ad689> in <module>()
          7 
          8 
    ----> 9 eggs()
    

    <ipython-input-8-52c0d52ad689> in eggs()
          3 
          4 def eggs():
    ----> 5     spam += 1
          6     print(spam)
          7 


    UnboundLocalError: local variable 'spam' referenced before assignment


在`eggs`函数内，如果某个变量有被赋值，那么该变量就是局部变量，如果没有赋值，那么便会被解释为全局变量，所以对局部变量`spam`进行操作的时候，在函数内并没有进行赋值，`spam`被解释成全局变量，自然会出现错误。

### 2.2.4.Modifying while iterating
当对某个字典进行迭代的时候：


```python
dict_ = {'spam': 'eggs'}

for key in dict_:
    del dict_[key]
```


    ---------------------------------------------------------------------------

    RuntimeError                              Traceback (most recent call last)

    <ipython-input-9-b88de85f5f0b> in <module>()
          1 dict_ = {'spam': 'eggs'}
          2 
    ----> 3 for key in dict_:
          4     del dict_[key]


    RuntimeError: dictionary changed size during iteration


改正方法如下：
```
dict_ = {'spam': 'eggs'}

for key in list(dict_):
    del dict_[key]
```
### 2.2.5.Late binding
这个问题就是所谓的后期绑定，什么是后期绑定？请看下面的例子：


```python
eggs = [lambda a: i * a for i in range(3)]
for egg in eggs:
    print(egg(5))
# 理想结果
# 0
# 5
# 10
```

    10
    10
    10


果然现实和理想是有差别的，为什么会造成这样的结果呢，是因为在匿名函数中`i`的值是2，可是明明分别循环了`0、1、2`。
你需要知道，定义一个函数后，函数内的变量并不是立刻就把值绑定了，而是在被调用的时候才会去查找变量，所以等`egg(5)`去查找i这个变量的时候，`i=5`，于是就造成了后期绑定这个陷阱。
要解决这个问题很简单，只需要在出现后期绑定前提前对变量进行*赋值*操作即可


```python
# 粗暴点可以直接这么来
eggs = [lambda a, i=i: i * a for i in range(3)]
for egg in eggs:
    print(egg(5))
```

    0
    5
    10



```python
# 优雅点
eggs = (lambda a: i * a for i in range(3))
for egg in eggs:
    print(egg(5))
```

    0
    5
    10



```python
# 高级语法这么来
import functools
eggs = [functools.partial(lambda i, a: i * a, i) for i in range(3)]
for egg in eggs:
    print(egg(5))
```

    0
    5
    10


总之，以上作为参考，能解决就好。

# 3.总结

本节笔记主要记录`python`一些优雅的写法以及陷阱。
