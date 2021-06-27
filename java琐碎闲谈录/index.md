# Java琐碎闲谈录


![](https://gitee.com/lonercci/picbed/raw/master/img/20210620111253.jpg)


> 琐碎闲谈录:
> 	记录一些零零散散，日常遇到的一些问题和知识点。



### 关于构造函数

在 Idea 中，如果有一个类，类中定义了几个属性，那么想要生成构造函数太简单了。快捷键一按，有参构造，无参构造，选中直接生成。
```java

public class Person {
    private Cat cat;
    private Dog dog;
    private String name;
	
    //无参构造
    public Person() {
        
    }
    
    //有参构造
    public Person(Cat cat, Dog dog, String name) {
        this.cat = cat;
        this.dog = dog;
        this.name = name;
    }
    
    //普通函数
    public void Method(){
        System.out.println("我太普通了....");
    }


}
```

这时，我们会发现，

与普通函数相比，构造出来的函数竟然不需要返回类型。

再仔细研究研究，

发现无参构造里面是空的，似乎我们可以往里面写内容？

事实上，**在 JVM 的运行机制里，如果我们在一个类中不声明构造函数， JVM 会帮我们默认生成一个空参数的构造函数（无参构造函数），我们把它称为 隐式构造。如果我们在类中声明了带参数列表的构造函数（有参构造函数），那么 JVM 就不会帮我们默认生成一个无参构造。所以，我们如果想要使用无参构造，那么就必须 显式的声明一个无参构造器。**（即在无参构造器里面自己写内容），这样上面的代码就变成了下面的：

```java
public Person() {
        System.out.println("这是一个无参构造器")
    }
```

紧接着问题又来了，这些构造函数有什么用？

+ 创建对象。任何一个对象创建时，都需要初始化才能使用，所以任何类想要创建实例对象就必须具有构造函数。
+ 对象初始化。构造函数可以对对象进行初始化，并且是给与之格式（参数列表）相符合的对象初始化，是具有一定针对性的初始化函数。

好家伙，我看不懂，但我大受震撼。**初始化是什么？为啥要初始化？**

java规定，变量没有初始化不能使用，全局变量也就是类的属性，java会在编译的时候，自动将他们初始化，所以可以不进行变量初始化的操作，但是（局部）变量必须初始化。**初始化就是在最开始定义成员变量时给它一个初始的值，为了防止程序运行时候出现未知的错误，或者bug，总之一句话，为了安全。**



>   闲谈：初学Java的时候，对这些理解迷迷糊糊的。后来在写程序的时候，还是迷迷糊糊的，经常会对这些基础的东西感到困惑，为什么要用到构造函数，构造函数究竟在代码中实现了怎样的功能。
>
>   文章参考：
>
>   [谈谈 java 中的构造函数](https://juejin.cn/post/6844903478234447886)
>
>   [Java构造函数，初始化，this关键字](https://zhuanlan.zhihu.com/p/102637985)







### 浅谈 Getter/Setter 方法

java 有一个不成文的规定，如果要访问一个类的 private 字段，就需要写 getter/setter 方法。但我们在其它语言却很少见到类似的约定，为什么？

-   它是“封装”的体现，对外隐藏了具体实现，允许之后对属性的访问注入新的逻辑（如验证逻辑）。
-   一些语言，如 python，提供了机制允许我们更改访问属性的逻辑，因此不需要手工写 getter/setter。

####  getter/setter 是对“属性访问”的封装

假设我们写了下面这段代码，直接访问类的 public 字段：

```java
class Person {
    public String name;
}

// caller
String name = person.name;
person.name = "Java";
```

之后我们认为 `name` 属性只能是字母，不能包含其它的字符，上面这种实现中，我们就需要更改所有 caller 调用 `person.name = ...` 的代码。换句话说，类 `Person` 暴露了实现的细节（即字段 person）。

那么如果一开始就使用了 getter/setter，则我们不需要改变任何 caller，只需要在 `setName` 函数里增加相应的逻辑即可。


```java

class Person {
    private String name;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        validate_name(name);  // the newly added validation logic
        this.name = name;
    }
}

// caller
String name = person.getName();
person.setName("Java");

```
所以，通过这层封装，之后如果有需要，我们甚至可以更改字段的名字，类型等等。这就是封装的好处，而 getter/setter 这种写法能让我们为将来可能的修改做好准备。

#### 其它语言里的 getter/setter

getter/setter 的作用是为“属性的访问”（即 `x.field` 与 `x.field = ...`）提供日后修改的可能。一些“比较新”的语言就默认提供了这种能力。

Python 中提供了 [Descriptor](https://docs.python.org/3/howto/descriptor.html) 的机制。在 Python 中，可以认为当访问对象的属性时，等价于调用对象的 `__get__()` 和 `__set__()` 方法，因此我们可以覆盖这两个方法来修改访问的逻辑。

同样的，Kotlin 在定义 [properties](https://kotlinlang.org/docs/reference/properties.html) 也可以自定义的 getter/setter 方法来修改属性访问的逻辑。

这里想说明的是，getter/setter 其实应该是默认实现，然后有需要时再覆盖，而不是每次都手工实现。

#### 社区与约定

也许你会问，封装其实叫什么名字都行，为什么非要叫 `getXXX` 及 `setXXX` 呢？这其实是 [JavaBeans](http://download.oracle.com/otn-pub/jcp/7224-javabeans-1.01-fr-spec-oth-JSpec/beans.101.pdf?AuthParam=1522674989_0d7c790344741da888ed8c0e890ea7d5) 里约定的（7.1 节）。甚至从某种角度来说 getter/setter 的目的也不是为了封装，而只是一个约定，使框架能识别 JavaBeans 中的 property。

在实际工作中你会发现，90% 以上的 getter/setter 在未来并不会被用来增加逻辑什么的。所以“封装”的作用理论上是好的，但实际被使用到的频率特别低，反而增加了许多无用的代码。

另一方面，随着使用 getter/setter 使用的增加，且由于绝大多数 getter/setter 并不会增加额外的逻辑，使得人们开始习惯于假设 getter/setter 不会有额外逻辑。所以如果你想在 setter 里加一些额外的逻辑时，反而要注意会不会让使用的人感到吃惊。

#### 写在最后

Getter/Setter 这个话题看上去似乎很简单，它的背后却有很多可以深究和思考的内容的。有人说 Getter 没关系，可怕的是 Setter；也有说现在lombok 这么方便，用 Getter/Setter 有利无害；也有人说尽量避免使用 Getter/Setter。这些观点背后都藏着一些软件的设计思维。例如怎样设计类的接口，如何实现封装，这些都是后续需要学习思考的内容。

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210620211420.png" style="zoom:80%;" />


>   闲谈：跟上面一样，学的时候不太懂，反正跟着视频敲就对了，也不知道有什么意义。后来get/set方法出现频率越来越高（学Spring的时候，控制反转），突然就有点明白了，隐隐约约跟bean有点关系，嗯，回过头重新记录一下。
>   文章参考：[为什么 java 中要写 getter/setter？](https://lotabout.me/2018/QQA-why-use-getters-and-setters/)  （直接搬运了，写的很好）





### 组合优于继承原则

面向对象编程中，有一条非常经典的设计原则，那就是：组合优于继承，多用组合少用继承。因为大多数继承都可以利用组合（composition）、接口、委托（delegation）三个技术手段来实现。从而降低继承的层次和复杂的继承关系。

> 闲谈：以前用过，但是从未思考过还有这个概念。在学静态代理的时候，有这样个例子，关于房东，租客，中介三者之间的关系，房东有Rent方法，中介这个代理想要使用Rent()，可以使用继承，但是不建议。组合可以实现更多的独属于代理这个类的方法。
>
> 文章参考 ：[为什么组合先于继承](https://blog.csdn.net/fuzhongmin05/article/details/108646872)  
