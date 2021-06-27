# 装饰器 迭代器 生成器


### 函数返回值
   `return`是函数结束的标志，即函数代码一旦运行到`return`会立即终止，并将`return`后的值当做本次运行的结果返回。
   1、返回None: 函数体内没有return
   2、返回一个值：return 值
   3、返回多个值：用逗号分隔开多个值，会被return返回成元组。
### 可变长度的参数
   可变长度指的是在调用函数时，传入的值（实参）的个数不固定。
   实参是用来为形参赋值的，所以针对溢出的实参必须有对应的形参来接收。
#### 可变长度的位置参数
   \*形参名：用来接收溢出的关键字参数，溢出的位置实参会被\*保存成元组的格式然后赋值给紧跟其后的形参名。
   约定俗成使用`*args`

```python
def foo(x,y,z=1,*args): #在最后一个形参名args前加*号
    print(x)
    print(y)
    print(z)
    print(args)
>>> foo(1,2,3,4,5,6,7)  #实参1、2、3按位置为形参x、y、z赋值，多余的位置实参4、5、6、7都被*接收，以元组的形式保存下来，赋值给args，即args=(4, 5, 6,7)

1
2
3
(4, 5, 6, 7)
```
 \*可以用在实参中，实参中带\*，打散成位置实参,再传值。类似于js中的解构赋值。
```python
#  *可以用在实参中，实参中带*，先*后的值打散成位置实参
def func(x,y,z):
    print(x,y,z)

func(*[11,22,33]) # func(11，22，33)
func(*[11,22]) # func(11，22)

#  形参与实参中都带*
def func(x,y,*args): # args=(3,4,5,6)
    print(x,y,args)

func(1,2,[3,4,5,6])
func(1,2,*[3,4,5,6]) # func(1,2,3,4,5,6)
func(*'hello') # func('h','e','l','l','o')

```

#### 可变长度的关键字参数
   \*\*形参名：用来接收溢出的关键字参数，溢出的位置实参会被\*保存成字典的格式然后赋值给紧跟其后的形参名。
   约定俗成使用`**kwargs`
```python
def foo(x,**kwargs): #在最后一个参数kwargs前加**
    print(x)        
    print(kwargs)   
 
foo(y=2,x=1,z=3) #溢出的关键字实参y=2，z=3都被**接收，以字典的形式保存下来，赋值给kwargs
1
{'z': 3, 'y': 2}
```
   \*\*可以用于实参中（\*\*后只能跟字典），实参中带有\*\*，先\*\*后的值打散成关键字实参。
```python
# I：**形参名：用来接收溢出的关键字实参，**会将溢出的关键字实参保存成字典格式，然后赋值给紧跟其后的形参名
#           **后跟的可以是任意名字，但是约定俗成应该是kwargs
def func(x,y,**kwargs):
    print(x,y,kwargs)

func(1,y=2,a=1,b=2,c=3)

# II: **可以用在实参中(**后跟的只能是字典)，实参中带**，先**后的值打散成关键字实参
def func(x,y,z):
    print(x,y,z)

func(*{'x':1,'y':2,'z':3}) # func('x','y','z')
func(**{'x':1,'y':2,'z':3}) # func(x=1,y=2,z=3)

# 错误
func(**{'x':1,'y':2,}) # func(x=1,y=2)
func(**{'x':1,'a':2,'z':3}) # func(x=1,a=2,z=3)

# III: 形参与实参中都带**
def func(x,y,**kwargs):
    print(x,y,kwargs)

func(y=222,x=111,a=333,b=444)
func(**{'y':222,'x':111,'a':333,'b':4444})
```
####  混用\*与\*\*
   \*args必须在\*\*kwargs之前。
   所有参数可任意组合使用，但定义顺序必须是：位置参数、默认参数、args、命名关键字参数、\*kwargs
   可变参数\*args与关键字参数kwargs通常是组合在一起使用的，**如果一个函数的形参为\*args与kwargs，那么代表该函数可以接收任何形式、任意长度的参数**

```python
def wrapper(*args,**kwargs):
    pass
```
   该函数内部还可以把接收到的参数传给另外一个函数。
```python
def func(x,y,z):
    print(x,y,z)
def wrapper(*args,**kwargs):
    func(*args,**kwargs)
wrapper(1,z=3,y=2)
1 2 3
```
   即在为函数`wrapper`传参时，其实遵循的是函数`func`的参数规则，调用函数`wrapper`的过程分析如下：
   位置实参`1`被接收，以元组的形式保存下来，赋值给`args`，即`args=(1,)`,关键字实参`z=3，y=2`被`*`接收，以字典的形式保存下来，赋值给`kwargs`，即`kwargs={'y': 2, 'z': 3}`
执行`func(args,kwargs)`,即`func((1,),* {'y': 2, 'z': 3})`,等同于`func(1,z=3,y=2)`。
   此方法后面装饰器会用到。
来源：
   `https://zhuanlan.zhihu.com/p/108907210` 
   2021.1.10

### 闭包函数
**前提条件：**
   闭包函数 = 名称空间与作用域+函数嵌套+函数对象。
   核心点：名字的查找关系是**以函数定义阶段为准**。
**什么是闭包函数：**
   闭：指该函数是内嵌函数。
   包：指的是该函数对外层函数作用域名字的引用（不是对全局作用域）。
**作用：**
   目前为止，我们得到了两种为函数体传值的方式，一种是直接将值以参数的形式传入，另外一种就是将值包给函数

```python
import requests

#方式一：
def get(url):
    return requests.get(url).text

#方式二：
def page(url):
    def get():
        return requests.get(url).text
    return get
```
   对比两种方式，方式一在下载同一页面时需要重复传入url，而方式二只需要传一次值，就会得到一个包含指定url的闭包函数，以后调用该闭包函数无需再传url。
```python
# 方式一下载同一页面
get('https://www.python.org')
get('https://www.python.org')
get('https://www.python.org')
……

# 方式二下载同一页面
python=page('https://www.python.org')
python()
python()
python()
……
```
闭包函数的这种特性有时又称为惰性计算。使用将值包给函数的方式，在接下来的装饰器中也将大有用处。

**个人总结：**
   通过以上案例可以看到，我们想要为一个函数体传参，有两种方式。
   一种为直接传参，定义的函数作用于全局变量，直接传入参数即可。
   而第二种很巧妙，把值包给函数。在全局变量下的函数`get()`上，再包一层函数`page()`，那么原函数`get()`就变成了一个内嵌函数。本来`get()`函数是无法与外界沟通的，但是通过`return`这个函数名，即返回此函数的内存地址，这样，通过外层函数的调用`page('https://www.python.org')`赋值给变量`python`。此变量的值就是内层函数的内存地址，加上`()`，即可调用内层函数，实现传参的目的。

资料来源：
1、`https://www.bilibili.com/video/BV1QE41147hU?p=232`
2、`https://zhuanlan.zhihu.com/p/109056932`

### 装饰器
   什么是装饰器：装饰器指的定义一个函数，该函数是用来为其他函数添加额外的功能。
   为什么要用装饰器：开放封闭原则
   - 开放：指的是对拓展功能是开放的
   - 封闭：指的是对修改源代码是封闭的
装饰器就是在不修改被装饰器对象源代码以及调用方式的情况下新的功能。

#### 无参装饰器
偷梁换柱之瞒天过海:
   原理：将原函数名指向的内存地址偷梁换柱成`wrapper`函数，再将`wrapper`函数的内存地址赋值给一个与原函数名相同的变量，加上`()`，就跟原函数调用一模一样的了。
   实现方法：
   1、首先是参数的传入，使用`*args,**kwargs`可以实现对任何个数的参数原封不动的通过新函数传入给原函数。
   2、但是，想实现对任意函数都能被装饰，那么就需要传入函数名，使用闭包函数的方法来实现，这样实现了原函数需要什么参数，通过传递就能给什么参数。
   3、若函数有返回值，就需要对返回值进行模拟。直接把原函数赋值给一个变量`res`,此变量接收原函数返回值，再到新函数里重新返回一次。至此，偷梁换柱完成。

```python
def outter(func):
    def wrapper(*args,**kwargs)
        res = func(*args,**kwargs)
        return res
    return wrapper
```
语法糖：`@名字`后面`def home()`==>`home=名字(home) `==>即把原先的`home()`内存地址替换为新的添加装饰器之后的内存地址。不用再写`home = outer(home)`了。

资料来源：
1、`https://www.bilibili.com/video/BV1QE41147hU?p=242`
2、`https://zhuanlan.zhihu.com/p/109078881`

#### 有参装饰器
   由于语法糖`@`的限制，`outter`函数只能有一个参数，并且该参数只能用来接收被装饰对象的内存地址。
   @名字(参数1，···)
```python
def 有参装饰器(x,y,z)
	def outter(func):
    	def wrapper(*args,**kwargs)
        	res = func(*args,**kwargs)
        	return res
    	return wrapper

@有参装饰器(1,y=2,z=3)
def 被装饰对象():
	pass
```
   至此，最多三层，就可以实现其余任意参数的传递。
#### 多个装饰器叠加分析
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210112231538.png" style="zoom: 50%;" />
加载顺序：自下而上
执行顺序：自上而下，即wrapper1==>wrapper2==>wrapper3

资料来源：

1、`https://www.bilibili.com/video/BV1QE41147hU?p=249`

2、`https://zhuanlan.zhihu.com/p/109078881`

3、`https://www.bilibili.com/video/BV1QE41147hU?p=257`


### 迭代器
   什么是迭代器：迭代器指的是迭代取值的工具，迭代是一个重复的过程，且每次重复都是基于上一次的结果而继续的，单纯的复制并不是迭代。
   为什么要有迭代器：通过索引的方式进行迭代取值，但仅适用于序列类型：字符串，列表，元组。对于没有索引的字典、集合等非序列类型，必须找到一种不依赖索引来进行迭代取值的方式，这就用到了迭代器。
   如何用迭代器：只要内置有`__iter__`方法的都可称之为可迭代对象。
#### for循环原理
   有了迭代器后，我们便可以不依赖索引迭代取值了，使用while循环的实现方式如下
```python
goods=['mac','lenovo','acer','dell','sony']
i=iter(goods) #每次都需要重新获取一个迭代器对象
while True:
    try:
        print(next(i))
    except StopIteration: #捕捉异常终止循环
        break
```
for循环又称为迭代循环，in后可以跟任意可迭代对象，上述while循环可以简写为
```python
goods=['mac','lenovo','acer','dell','sony']
for item in goods:   
    print(item)
```
for 循环在工作时，首先会调用可迭代对象goods内置的iter方法拿到一个迭代器对象，然后再调用该迭代器对象的next方法将取到的值赋给item,执行循环体完成一次循环，周而复始，直到捕捉StopIteration异常，结束迭代。

#### 迭代器的优缺点
优点：
- 为序列和非序列类型提供了一种统一的迭代取值方式。
- 惰性计算：迭代器对象表示的是一个数据流，可以只在需要时才去调用next来计算出一个值，就迭代器本身来说，同一时刻在内存中只有一个值，因而可以存放无限大的数据流，而对于其他容器类型，如列表，需要把所有的元素都存放于内存中，受内存大小的限制，可以存放的值的个数是有限的。

缺点：
- 除非取尽，否则无法获取迭代器的长度
- 只能取下一个值，不能回到开始，更像是‘一次性的’，迭代器产生后的唯一目标就是重复执行next方法直到值取尽，否则就会停留在某个位置，等待下一次调用next；若是要再次迭代同个对象，你只能重新调用iter方法去创建一个新的迭代器对象，如果有两个或者多个循环使用同一个迭代器，必然只会有一个循环能取到值。

### 生成器
若函数体包含`yield`关键字，再调用函数，并不会执行函数体代码，得到的返回值即生成器对象
```python
>>> def my_range(start,stop,step=1):
        print('start...')
        while start < stop:
            yield start
            start+=step
        print('end...')
        
>>> g=my_range(0,3)
>>> g
<generator object my_range at 0x104105678>
```
生成器内置有`__iter__`和`__next__`方法，所以生成器本身就是一个迭代器
```python
>>> g.__iter__
<method-wrapper '__iter__' of generator object at 0x1037d2af0>
>>> g.__next__
<method-wrapper '__next__' of generator object at 0x1037d2af0>
```
因而我们可以用`next(生成器)`触发生成器所对应函数的执行，
```python
>>> next(g) # 触发函数执行直到遇到yield则停止,将yield后的值返回，并在当前位置挂起函数
start...
0
>>> next(g) # 再次调用next(g)，函数从上次暂停的位置继续执行，直到重新遇到yield...
1
>>> next(g) # 周而复始...
2
>>> next(g) # 触发函数执行没有遇到yield则无值返回，即取值完毕抛出异常结束迭代
end...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
既然生成器对象属于迭代器，那么必然可以使用`for`循环迭代，如下
```python
>>> for i in countdown(3):
        print(i)
    
countdown start
3
2
1
Done!
```
有了`yield`关键字，我们就有了一种自定义迭代器的实现方式。`yield`可以用于返回值，但不同于`return`，函数一旦遇到`return`就结束了，而`yield`可以保存函数的运行状态挂起函数，用来返回多次值。`next`用来触发函数运行。


