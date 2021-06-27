# Js混淆-动态cookie


## 题目
![](https://gitee.com/lonercci/picbed/raw/master/img/20210127222126.png)

题目链接：http://match.yuanrenxue.com/match/2

> 提取全部5页发布日热度的值，计算所有值的加和,并提交答案

## 分析网页
本题题目已经写说明了是动态cookie，所以为了避免其余cookie的影响，直接在无痕模式下进行调试。

老样子，试着看看XHR里面的内容。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210127222633.png" style="zoom: 50%;" />
发现跟上一道题一样，数据都是在json里面，看下后面几页的网址。  
第一页：http://match.yuanrenxue.com/api/match/2 点击访问一下看看。  不出所料，返回的不是正常的内容，而是`{"error": "出错啦~嘤嘤嘤"}`。我们观察一下请求的cookie，貌似跟第一题的加密方法很像，`|`后面应该是个时间戳。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210127234155.png" style="zoom:50%;" />
那么这个cookie是怎么生成的呢?

## 寻找cookie来源
查看所有的请求信息，看看有没有别的东西。发现有两个请求都是`2`，有点可疑，点进去看看
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210127235352.png" style="zoom:50%;" />

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210127235540.png" style="zoom:50%;" />

可以发现,第一个请求是不带cookie的,而第二个请求带cookie，那么我们是不是可以猜测第一个请求完成后，执行了某些我们没有看到的操作，从而生成了cookie，给了第二个请求。

## 使用Fiddler抓包
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128000459.png" style="zoom: 67%;" />

我们可以看到，第一次请求确实是返回了一些东西。是一段JS代码，但是这些看着全是乱七八糟的字符。这说明这段代码经过了一定的混淆，先不管其他，我们把它复制粘贴下来，格式化一下，
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128000948.png" style="zoom: 50%;" />
代码太长了，就截取一部分。查阅资料，

> 混淆后的样子，变量跟方法名已经完全替换，然后第一行可以看到明显的一个数组，这就是ob混淆的常见特性.

而官网正好就有 [解混淆工具](http://tool.yuanrenxue.com/decode_obfuscator) 我们把代码解混淆之后的代码继续格式化
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128001721.png" style="zoom: 50%;" />
整段代码差不多200多行代码,而且这些函数名和变量名都非常奇怪。后来查阅资料才知道，这是由于经过一些工具压缩了JS之后导致的，简化了变量名和方法名便于传输。那么我们下一步就很明确了，要分析这段js代码到底干了什么。

## 分析JS代码
大致的浏览一下,全部代码,可以发现,大多数都是定义的函数,但是真正执行的只有最后的`W(X());`
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128002158.png" style="zoom:50%;" />

我们首先看看X()函数是什么
```js
  function X(Y, Z) {
    return Date["parse"](new Date());
  }
```
在试工具里面执行X()函数就可以看到,返回的是一个时间戳,那么这个函数也会返回一个时间戳
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128002615.png" style="zoom: 67%;" />

再来看W()函数是什么
```js
  function W(Y, Z) {
    document["cookie"] = "m" + M() + "=" + V(Y) + "|" + Y + "; path=/";
    location["reload"]();
  }
```
w()函数定义了两个形参,而执行的函数`W(X());`,是传入一个参数的，发现W()里面包含cookie等信息，那么肯定要研究一下W()干了什么

## 分析W()函数
我们来看一下W()函数的作用
+ `document["cookie"] `就是设置cookie信息
+ `location["reload"](); `非常关键！！这行代码的意思就是:刷新当前文档，也就是按了一下浏览器上的刷新页面按钮。
这段代码解释了前面为何会出现两个相同的请求。我们整理一下。  
1. 网页向服务器发送请求,返回了两个名为【2】的结果
2. 第一个没有Cookie,而第二个有Cookie
3. 第一个虽然没有Cookie,但是却执行了一段JS代码
4. 而这段JS代码给网页中的Cookie赋了值,接着刷新了整个页面
5. 最后,呈现在我们眼前的网页,也就是第二个名为【2】的结果,有了Cookie

那么现在就是关键的`document["cookie"]`部分，
```js
m=36674c3718305e203ee914bf011045c5|1609132022000
document["cookie"] = "m" + M() + "=" + V(Y) + "|" + Y + "; path=/";
```
最后面的`"; path=/"`字符串,是页面信息(第几页),在其他地方肯定是会处理的,这里我们可以省略。
通过对比,赋值过程和赋值结果,我们可以发现赋值过程中的` M()`函数返回的一个应该是个空值。但是我们可以去稍微分析一下M()函数。
```js
function M(Y, Z) {
        var a4 = B(this, function () {
            var a5 = function () {
                var a6 = a5["constructor"]("return /\" + this + \"/")()["compile"]("^([^ ]+( +[^ ]+)+)+[^ ]}");
                return !a6["test"](a4);
            };

            return a5();
        });
        a4();
        K();
        qz = [10, 99, 111, 110, 115, 111, 108, 101, 32, 61, 32, 110, 101, 119, 32, 79, 98, 106, 101, 99, 116, 40, 41, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 32, 61, 32, 102, 117, 110, 99, 116, 105, 111, 110, 32, 40, 115, 41, 32, 123, 10, 32, 32, 32, 32, 119, 104, 105, 108, 101, 32, 40, 49, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 102, 111, 114, 40, 105, 61, 48, 59, 105, 60, 49, 49, 48, 48, 48, 48, 48, 59, 105, 43, 43, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 104, 105, 115, 116, 111, 114, 121, 46, 112, 117, 115, 104, 83, 116, 97, 116, 101, 40, 48, 44, 48, 44, 105, 41, 10, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 125, 10, 32, 32, 32, 32, 125, 10, 10, 125, 10, 99, 111, 110, 115, 111, 108, 101, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 91, 111, 98, 106, 101, 99, 116, 32, 79, 98, 106, 101, 99, 116, 93, 39, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 402, 32, 116, 111, 83, 116, 114, 105, 110, 103, 40, 41, 32, 123, 32, 91, 110, 97, 116, 105, 118, 101, 32, 99, 111, 100, 101, 93, 32, 125, 39, 10];
        eval(L(qz));

        try {
            if (global) {
                console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
            } else {
                while (1) {
                    console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
                    debugger;
                }
            }
        } catch (a5) {
            return navigator["vendorSub"];
        }
    }
```
首先,我们可以看到函数里面还包含着两个函数分别是【 a4() 】和【 K() 】
而这个【 a4() 】函数,也定义在M()内,但是执行【 a4() 】函数的时候,并没有传入参数,所以说,这段代码是没用的

接着,我们看一下K()函数
```js
function K(Y, Z) {
        if (Z) {
            return J(Y);
        }

        return H(Y);
    }
```
需要传入参数,而执行的时候,又没有传入,所有这段代码也是是没用的
我们简化一下M()函数
```js
function M(Y, Z) {
        qz = [10, 99, 111, 110, 115, 111, 108, 101, 32, 61, 32, 110, 101, 119, 32, 79, 98, 106, 101, 99, 116, 40, 41, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 32, 61, 32, 102, 117, 110, 99, 116, 105, 111, 110, 32, 40, 115, 41, 32, 123, 10, 32, 32, 32, 32, 119, 104, 105, 108, 101, 32, 40, 49, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 102, 111, 114, 40, 105, 61, 48, 59, 105, 60, 49, 49, 48, 48, 48, 48, 48, 59, 105, 43, 43, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 104, 105, 115, 116, 111, 114, 121, 46, 112, 117, 115, 104, 83, 116, 97, 116, 101, 40, 48, 44, 48, 44, 105, 41, 10, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 125, 10, 32, 32, 32, 32, 125, 10, 10, 125, 10, 99, 111, 110, 115, 111, 108, 101, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 91, 111, 98, 106, 101, 99, 116, 32, 79, 98, 106, 101, 99, 116, 93, 39, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 402, 32, 116, 111, 83, 116, 114, 105, 110, 103, 40, 41, 32, 123, 32, 91, 110, 97, 116, 105, 118, 101, 32, 99, 111, 100, 101, 93, 32, 125, 39, 10];
        eval(L(qz));

        try {
            if (global) {
                console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
            } else {
                while (1) {
                    console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
                    debugger;
                }
            }
        } catch (a5) {
            return navigator["vendorSub"];
        }
    }
```
eval()函数又出现了,做过第一道题的时候,我们知道eval()可以间接的改变一些变量的值,那么这里的eval()是否对cookie有所改变呢?
我们来看看eval()中的L()函数

```js
function L(Y, Z) {
        let a0 = "";

        for (let a1 = 0; a1 < Y["length"]; a1++) {
            a0 += String["fromCharCode"](Y[a1]);
        }

        return a0;
    }
```
L()函数可以传入两个参数,eval()执行的时候确实是传入一个参数的,这个参数是一个数组,正好用于L()函数中的for循环
for循环里面的内容就是将,传入的数组中的每一个值,通过【 String["fromCharCode"] 】这个方法翻译拼接着成一个字符串并返回
尝试着运行了这个函数,结果并没有什么卵用,就是一串字符串而已,这种就是典型的开发者挖坑,我们去踩

我们继续读下面的代码
```js
        try {
            if (global) {
                console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
            } else {
                while (1) {
                    console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
                    debugger;
                }
            }
        } catch (a5) {
            return navigator["vendorSub"];
        }
```
try里面的内容,直接进来就报错。是因为,global这个变量根本就没有定义,所有会直接报错,进入catch里面,虽然说传入了a5变量,但是在下面的代码并没有引用,可以不管
最后,直接return navigator["vendorSub"]
而这个navigator主要是包含有关浏览器的信息,它通过数组取值的方式取值,大概率返回的任然是空值.如果不是我们可以回来继续分析

判断M()八成是空值，于是我们可以吧代码简化一下
```js
cookie = "m" + "=" + V(Y) + "|" + Y;
          m=36674c3718305e203ee914bf011045c5|1609132022000
V(Y) = 36674c3718305e203ee914bf011045c5
```
我们现在就知道了,加密的值来源于`V(Y)`
`V(Y)`具体怎么实现的，我们可以直接先验证一下，看能不能得到结果。

## 验证猜测
我们要改写JS，做出以下调整。
- 删除第一行代码和最后一行代码，去掉最外面一层，方便调试
- 发现定义时变量报错，把循环内部局部变量改成全局的，即代码中的let改为var
- 尝试运行时是会某明奇妙的卡住，退出。删除104行左右的`setInterval(M(), 500);`这行代码,而这行代码的主要作用就是:每500毫秒,执行M(),而这个函数,我们在上面的推断是返回空所有没什么用,可以直接删掉,否则会一直卡住。
- 显示未定义navigator，那么在代码的开头定义 `var navigator = {}; `（这个navigator包含有关浏览器的信息）
- M()函数中,qz上面的多余代码直接删除,并没有什么卵用
- 改写W()函数中的内容,添加新的函数`get_chipher`执行结果
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128112158.png" style="zoom: 50%;" />

我们可以看到，运行之后拿到了值`m=ac27a113166d13b2d6964bd9657f9020|1611801060000`
再看看原本的cookie是什么
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128112553.png" style="zoom: 50%;" />

比对成功，我们把js文件那个时间戳改成正常的调用方法
```js
function X(Y, Z) {
    return Date["parse"](new Date());
}
```
这样，我们对JS部分解决完毕。此处，附上JS部分的代码：
```js
var navigator = {};
var B = function () {
    var Y = true;
    return function (Z, a0) {
        var a1 = Y ?
            function () {
                if (a0) {
                    var a2 = a0["apply"](Z, arguments);
                    a0 = null;
                    return a2;
                }
            } : function () {
            };
        Y = false;
        return a1;
    };
}();

function C(Y, Z) {
    var a0 = (65535 & Y) + (65535 & Z);
    return (Y >> 16) + (Z >> 16) + (a0 >> 16) << 16 | 65535 & a0;
}

function D(Y, Z) {
    return Y << Z | Y >>> 32 - Z;
}

function E(Y, Z, a0, a1, a2, a3) {
    return C(D(C(C(Z, Y), C(a1, a3)), a2), a0);
}

function F(Y, Z, a0, a1, a2, a3, a4) {
    return E(Z & a0 | ~Z & a1, Y, Z, a2, a3, a4);
}

function G(Y, Z, a0, a1, a2, a3, a4) {
    return E(Z & a1 | a0 & ~a1, Y, Z, a2, a3, a4);
}

function H(Y, Z) {
    var a0 = [99, 111, 110, 115, 111, 108, 101];
    var a1 = "";

    for (var a2 = 0; a2 < a0["length"]; a2++) {
        a1 += String["fromCharCode"](a0[a2]);
    }

    return a1;
}

function I(Y, Z, a0, a1, a2, a3, a4) {
    return E(Z ^ a0 ^ a1, Y, Z, a2, a3, a4);
}

function J(Y, Z, a0, a1, a2, a3, a4) {
    return E(a0 ^ (Z | ~a1), Y, Z, a2, a3, a4);
}

function K(Y, Z) {
    if (Z) {
        return J(Y);
    }

    return H(Y);
}

function L(Y, Z) {
    var a0 = "";

    for (var a1 = 0; a1 < Y["length"]; a1++) {
        a0 += String["fromCharCode"](Y[a1]);
    }

    return a0;
}

function M(Y, Z) {
    qz = [10, 99, 111, 110, 115, 111, 108, 101, 32, 61, 32, 110, 101, 119, 32, 79, 98, 106, 101, 99, 116, 40, 41, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 32, 61, 32, 102, 117, 110, 99, 116, 105, 111, 110, 32, 40, 115, 41, 32, 123, 10, 32, 32, 32, 32, 119, 104, 105, 108, 101, 32, 40, 49, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 102, 111, 114, 40, 105, 61, 48, 59, 105, 60, 49, 49, 48, 48, 48, 48, 48, 59, 105, 43, 43, 41, 123, 10, 32, 32, 32, 32, 32, 32, 32, 32, 104, 105, 115, 116, 111, 114, 121, 46, 112, 117, 115, 104, 83, 116, 97, 116, 101, 40, 48, 44, 48, 44, 105, 41, 10, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 125, 10, 32, 32, 32, 32, 125, 10, 10, 125, 10, 99, 111, 110, 115, 111, 108, 101, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 91, 111, 98, 106, 101, 99, 116, 32, 79, 98, 106, 101, 99, 116, 93, 39, 10, 99, 111, 110, 115, 111, 108, 101, 46, 108, 111, 103, 46, 116, 111, 83, 116, 114, 105, 110, 103, 32, 61, 32, 39, 402, 32, 116, 111, 83, 116, 114, 105, 110, 103, 40, 41, 32, 123, 32, 91, 110, 97, 116, 105, 118, 101, 32, 99, 111, 100, 101, 93, 32, 125, 39, 10];
    eval(L(qz));

    try {
        if (global) {
            console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
        } else {
            while (1) {
                console["log"]("\u4EBA\u751F\u82E6\u77ED\uFF0C\u4F55\u5FC5python\uFF1F");
                debugger;
            }
        }
    } catch (a5) {
        return navigator["vendorSub"];
    }
}

function N(Y, Z) {
    Y[Z >> 5] |= 128 << Z % 32,
        Y[14 + (Z + 64 >>> 9 << 4)] = Z;

    if (qz) {
        var a0, a1, a2, a3, a4, a5 = 1732584193,
            a6 = -271733879,
            a7 = -1732584194,
            a8 = 271733878;
    } else {
        var a0, a1, a2, a3, a4, a5 = 0,
            a6 = -0,
            a7 = -0,
            a8 = 0;
    }

    for (a0 = 0; a0 < Y["length"]; a0 += 16) a1 = a5,
        a2 = a6,
        a3 = a7,
        a4 = a8,
        a5 = F(a5, a6, a7, a8, Y[a0], 7, -680876936),
        a8 = F(a8, a5, a6, a7, Y[a0 + 1], 12, -389564586),
        a7 = F(a7, a8, a5, a6, Y[a0 + 2], 17, 606105819),
        a6 = F(a6, a7, a8, a5, Y[a0 + 3], 22, -1044525330),
        a5 = F(a5, a6, a7, a8, Y[a0 + 4], 7, -176418897),
        a8 = F(a8, a5, a6, a7, Y[a0 + 5], 12, 1200080426),
        a7 = F(a7, a8, a5, a6, Y[a0 + 6], 17, -1473231341),
        a6 = F(a6, a7, a8, a5, Y[a0 + 7], 22, -45705983),
        a5 = F(a5, a6, a7, a8, Y[a0 + 8], 7, 1770010416),
        a8 = F(a8, a5, a6, a7, Y[a0 + 9], 12, -1958414417),
        a7 = F(a7, a8, a5, a6, Y[a0 + 10], 17, -42063),
        a6 = F(a6, a7, a8, a5, Y[a0 + 11], 22, -1990404162),
        a5 = F(a5, a6, a7, a8, Y[a0 + 12], 7, 1804603682),
        a8 = F(a8, a5, a6, a7, Y[a0 + 13], 12, -40341101),
        a7 = F(a7, a8, a5, a6, Y[a0 + 14], 17, -1502882290),
        a6 = F(a6, a7, a8, a5, Y[a0 + 15], 22, 1236535329),
        a5 = G(a5, a6, a7, a8, Y[a0 + 1], 5, -165796510),
        a8 = G(a8, a5, a6, a7, Y[a0 + 6], 9, -1069501632),
        a7 = G(a7, a8, a5, a6, Y[a0 + 11], 14, 643717713),
        a6 = G(a6, a7, a8, a5, Y[a0], 20, -373897302),
        a5 = G(a5, a6, a7, a8, Y[a0 + 5], 5, -701558691),
        a8 = G(a8, a5, a6, a7, Y[a0 + 10], 9, 38016083),
        a7 = G(a7, a8, a5, a6, Y[a0 + 15], 14, -660478335),
        a6 = G(a6, a7, a8, a5, Y[a0 + 4], 20, -405537848),
        a5 = G(a5, a6, a7, a8, Y[a0 + 9], 5, 568446438),
        a8 = G(a8, a5, a6, a7, Y[a0 + 14], 9, -1019803690),
        a7 = G(a7, a8, a5, a6, Y[a0 + 3], 14, -187363961),
        a6 = G(a6, a7, a8, a5, Y[a0 + 8], 20, 1163531501),
        a5 = G(a5, a6, a7, a8, Y[a0 + 13], 5, -1444681467),
        a8 = G(a8, a5, a6, a7, Y[a0 + 2], 9, -51403784),
        a7 = G(a7, a8, a5, a6, Y[a0 + 7], 14, 1735328473),
        a6 = G(a6, a7, a8, a5, Y[a0 + 12], 20, -1926607734),
        a5 = I(a5, a6, a7, a8, Y[a0 + 5], 4, -378558),
        a8 = I(a8, a5, a6, a7, Y[a0 + 8], 11, -2022574463),
        a7 = I(a7, a8, a5, a6, Y[a0 + 11], 16, 1839030562),
        a6 = I(a6, a7, a8, a5, Y[a0 + 14], 23, -35309556),
        a5 = I(a5, a6, a7, a8, Y[a0 + 1], 4, -1530992060),
        a8 = I(a8, a5, a6, a7, Y[a0 + 4], 11, 1272893353),
        a7 = I(a7, a8, a5, a6, Y[a0 + 7], 16, -155497632),
        a6 = I(a6, a7, a8, a5, Y[a0 + 10], 23, -1094730640),
        a5 = I(a5, a6, a7, a8, Y[a0 + 13], 4, 681279174),
        a8 = I(a8, a5, a6, a7, Y[a0], 11, -358537222),
        a7 = I(a7, a8, a5, a6, Y[a0 + 3], 16, -722521979),
        a6 = I(a6, a7, a8, a5, Y[a0 + 6], 23, 76029189),
        a5 = I(a5, a6, a7, a8, Y[a0 + 9], 4, -640364487),
        a8 = I(a8, a5, a6, a7, Y[a0 + 12], 11, -421815835),
        a7 = I(a7, a8, a5, a6, Y[a0 + 15], 16, 530742520),
        a6 = I(a6, a7, a8, a5, Y[a0 + 2], 23, -995338651),
        a5 = J(a5, a6, a7, a8, Y[a0], 6, -198630844),
        a8 = J(a8, a5, a6, a7, Y[a0 + 7], 10, 1126891415),
        a7 = J(a7, a8, a5, a6, Y[a0 + 14], 15, -1416354905),
        a6 = J(a6, a7, a8, a5, Y[a0 + 5], 21, -57434055),
        a5 = J(a5, a6, a7, a8, Y[a0 + 12], 6, 1700485571),
        a8 = J(a8, a5, a6, a7, Y[a0 + 3], 10, -1894986606),
        a7 = J(a7, a8, a5, a6, Y[a0 + 10], 15, -1051523),
        a6 = J(a6, a7, a8, a5, Y[a0 + 1], 21, -2054922799),
        a5 = J(a5, a6, a7, a8, Y[a0 + 8], 6, 1873313359),
        a8 = J(a8, a5, a6, a7, Y[a0 + 15], 10, -30611744),
        a7 = J(a7, a8, a5, a6, Y[a0 + 6], 15, -1560198380),
        a6 = J(a6, a7, a8, a5, Y[a0 + 13], 21, 1309151649),
        a5 = J(a5, a6, a7, a8, Y[a0 + 4], 6, -145523070),
        a8 = J(a8, a5, a6, a7, Y[a0 + 11], 10, -1120210379),
        a7 = J(a7, a8, a5, a6, Y[a0 + 2], 15, 718787259),
        a6 = J(a6, a7, a8, a5, Y[a0 + 9], 21, -343485441),
        a5 = C(a5, a1),
        a6 = C(a6, a2),
        a7 = C(a7, a3),
        a8 = C(a8, a4);

    return [a5, a6, a7, a8];
}

function O(Y) {
    var Z, a0 = "",
        a1 = 32 * Y["length"];

    for (Z = 0; Z < a1; Z += 8) a0 += String["fromCharCode"](Y[Z >> 5] >>> Z % 32 & 255);

    return a0;
}

function P(Y) {
    var Z, a0 = [];

    for (a0[(Y["length"] >> 2) - 1] = undefined, Z = 0; Z < a0["length"]; Z += 1) a0[Z] = 0;

    var a1 = 8 * Y["length"];

    for (Z = 0; Z < a1; Z += 8) a0[Z >> 5] |= (255 & Y["charCodeAt"](Z / 8)) << Z % 32;

    return a0;
}

function Q(Y) {
    return O(N(P(Y), 8 * Y["length"]));
}

function R(Y) {
    var Z, a0, a1 = "0123456789abcdef",
        a2 = "";

    for (a0 = 0; a0 < Y["length"]; a0 += 1) Z = Y["charCodeAt"](a0),
        a2 += a1["charAt"](Z >>> 4 & 15) + a1["charAt"](15 & Z);

    return a2;
}

function S(Y) {
    return unescape(encodeURIComponent(Y));
}

function T(Y) {
    return Q(S(Y));
}

function U(Y) {
    return R(T(Y));
}

function V(Y, Z, a0) {
    M();
    return Z ? a0 ? H(Z, Y) : y(Z, Y) : a0 ? T(Y) : U(Y);
}

function W(Y, Z) {
    var cookie = "m" + "=" + V(Y) + "|" + Y;
    return cookie
}

function X(Y, Z) {
    return Date["parse"](new Date());
}

function get_chipher() {
    return W(X());
}
```
## Python爬取
加密部分解决，爬取部分就简单了。代码如下：
```python
import requests
import execjs

def get_cookies():
    with open(r'1.1.js',encoding='utf-8',mode='r') as f:
        JsData = f.read()
        ck= execjs.compile(JsData).call('get_chipher')
        print(f'cookie是{ck}')
        return ck

def get_data(url,cookie):
    headers={
        'user-agent':'yuanrenxue.project',
        'cookie':cookie
    }
    r = requests.get(url,headers=headers)
    return r.json()

if __name__ == '__main__':
    sum_all = 0

    for i in range(1,6):
        url = f'http://match.yuanrenxue.com/api/match/2?page={i}'
        info = get_data(url,get_cookies())
        hot_list =[i["value"] for i in info["data"]]
        print(f'第{i}页的数据为{hot_list}')
        sum_all += sum(hot_list)
    print(f'所有页数的热度加和为{sum_all}')

```
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128122236.png" style="zoom: 50%;" />

## 总结
本题的思路跟上一题稍有不同，上一题加密的部分是可以直接在js里面找到的，而本题却是在发送第一个请求的时候返回了一段ob混淆过的js代码。
在谷歌浏览器自带的调试工具里，某些操作是看不到的，比如说刚刚的那段JS。使用fiddler可以抓到包，以前安装过fiddler，但是不怎么会用，现在发现还挺香的，嘿嘿。
思路不难，跟第一题非常类似，分析JS。比较头疼的是JS的调试部分，开发者绕了很多圈子，写了不少类似死循环的东西误导你，拿到调试工具里面，一会就卡住了，所以对那些没有用的代码，精简删除很关键。
