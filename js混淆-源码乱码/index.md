# Js混淆-源码乱码


##  题目
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210124182912.png" style="zoom:50%;" />

> 抓取所有（5页）机票的价格，并计算所有机票价格的平均值，填入答案。

题目链接：http://match.yuanrenxue.com/match/1

## 思路分析

### 关闭断点
看着好像不难，但是打开F12的时候，会出现这个页面，发现无法查看数据。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210124183340.png" style="zoom:50%;" />

这是debug断点，直接关闭，继续运行。

### 查看数据
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210124185139.png" style="zoom:50%;" />
发现数据以json格式存放，那么我们只要拿到这个url地址，就能拿到所有数据

### 分析url
把这几页的url都拿出来看看
> http://match.yuanrenxue.com/api/match/1?m=5b6658c234a4d3b9a334bb7a652c6483%E4%B8%A81611585722
> http://match.yuanrenxue.com/api/match/1?page=2&m=6ab86178933c0ff456e01ebb40a9e9e1%E4%B8%A81611585928
> http://match.yuanrenxue.com/api/match/1?page=3&m=c531e3e5f28835032f9da3fc4e88718d%E4%B8%A81611585938

参数很明显了，一共有两个参数。page是页码，m是一个未知参数。
`%E4%B8%A8`是url转码后的东西，解码出来是`丨`（一个中文符号）。后面的东西会随着时间推移改变，根据经验也知道这一定是个时间戳。于是现在就是要分析前面的一堆字符是什么，看样子是经过加密后的东西。   
但是！！！经过测试发现在不同的时间段，同一个数据的url都不一样，时间戳改变可以理解，那为什么加密的部分也会改变呢？我们是否可以猜测，前面加密部分的动态改变，有时间戳的参与的可能？

### 如何寻找m的加密过程--->|
当时我就卡在这里了，如何去找到m参数的加密过程。尝试直接搜索m是行不通的，因为这个文件里m太多了。后来猜测m可能是由三部分拼接的，加密字符串+|+时间戳。那么在这个过程中，`|`就至关重要了。试着搜索看看，果然有惊喜，找到了一段js。我们把这段js复制下来，格式化以后，删除后面的ajax部分，最终得到如下代码：
```js
window.url = '/api/match/1';
request = function() {
var timestamp = Date.parse(new Date()) + 100000000;
var m = oo0O0(timestamp.toString()) + window.f;
var list = {
    "page ": window.page,
    "m ": m + '丨' + timestamp / 1000
};
request()
```
到这里似乎说不通了，我当时也有点懵，实际上 `"m ": m + '丨' + timestamp / 1000`只是list里面的一个参数的值，而我们这里需要的是变量m，即list里面的值与我们想要的无关，而`m = oo0O0(timestamp.toString()) + window.f`正好印证了我们刚刚的猜测，前面加密部分的字符串有时间戳的参与，于是继续精简
```js
var timestamp = Date.parse(new Date()) + 100000000;
var m = oo0O0(timestamp.toString()) + window.f;
```
可得所需要的m参数是由以上两行代码完成的，此时可以大胆的猜想一下，timestamp应该是一个时间戳，timestamp.toString()应该是一个转换为字符串的函数，传入oo0O0()中，一完成了某些操作，再与window.f的值相加。  
于是我们的突破口来了，分别是oo0O0()函数和window.f 。

### 进一步跟踪
window.f 窗口这个对象调用了f,至于f是什么东西,我们先不管。先来看oo0O0()函数。发现还是在上面搜索时的地方，我们把定义这个函数的部分截取下来。
```js
function oo0O0(mw) {
    window.b = '';
    for (var i = 0, len = window.a.length; i < len; i++) {
        console.log(window.a[i]);
        window.b += String[document.e + document.g](window.a[i][document.f + document.h]() - i - window.c)
    }
    var U = ['W5r5W6VdIHZcT8kU', 'WQ8CWRaxWQirAW=='];
    var J = function (o, E) {
        o = o - 0x0;
        var N = U[o];
        if (J['bSSGte'] === undefined) {
            var Y = function (w) {
                var m = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=',
                    T = String(w)['replace'](/=+$/, '');
                var A = '';
                for (var C = 0x0, b, W, l = 0x0; W = T['charAt'](l++); ~W && (b = C % 0x4 ? b * 0x40 + W : W, C++ % 0x4) ? A += String['fromCharCode'](0xff & b >> (-0x2 * C & 0x6)) : 0x0) {
                    W = m['indexOf'](W)
                }
                return A
            };
            var t = function (w, m) {
                var T = [], A = 0x0, C, b = '', W = '';
                w = Y(w);
                for (var R = 0x0, v = w['length']; R < v; R++) {
                    W += '%' + ('00' + w['charCodeAt'](R)['toString'](0x10))['slice'](-0x2)
                }
                w = decodeURIComponent(W);
                var l;
                for (l = 0x0; l < 0x100; l++) {
                    T[l] = l
                }
                for (l = 0x0; l < 0x100; l++) {
                    A = (A + T[l] + m['charCodeAt'](l % m['length'])) % 0x100, C = T[l], T[l] = T[A], T[A] = C
                }
                l = 0x0, A = 0x0;
                for (var L = 0x0; L < w['length']; L++) {
                    l = (l + 0x1) % 0x100, A = (A + T[l]) % 0x100, C = T[l], T[l] = T[A], T[A] = C, b += String['fromCharCode'](w['charCodeAt'](L) ^ T[(T[l] + T[A]) % 0x100])
                }
                return b
            };
            J['luAabU'] = t, J['qlVPZg'] = {}, J['bSSGte'] = !![]
        }
        var H = J['qlVPZg'][o];
        return H === undefined ? (J['TUDBIJ'] === undefined && (J['TUDBIJ'] = !![]), N = J['luAabU'](N, E), J['qlVPZg'][o] = N) : N = H, N
    };
    eval(atob(window['b'])[J('0x0', ']dQW')](J('0x1', 'GTu!'), '\x27' + mw + '\x27'));
    return ''
}
```
代码有点长，压缩一部分，
![](https://gitee.com/lonercci/picbed/raw/master/img/20210124223349.png)
很amazing啊，清晰的看到返回值居然是空字符串。这意味着 [ var m = window.f ]，那我们只能很不幸的得到以下这个结论：

```js
var m = oo0O0(timestamp.toString()) + window.f;   //之前
var m = window.f   //之后
```
可是`window.f`是什么东西啊

### 寻找window.f -->eval()
怎么找到`window.f`，首先我能想到的,就是去源码中搜索,但是结果并没有那么理想,只出现了一处代码,也就是刚刚那一处代码

在源码中找不到,但是又使用了`window.f`，肯定是通过间接的方式定义赋值的,那么它又在在那里定义的呢?
我们再来看一下这行代码
```js
var m = oo0O0(timestamp.toString()) + window.f;
```
oo0O0函数执行了,但是却返回的空值;而且oo0O0()里面却有很多代码,开发者肯定不会这么闲吧。也不排除,开发者写这些没有用的代码误导我们,但是这里面肯定是有内容的。
这就涉及到我的盲区了。。。
JS中的 eval() 函数可计算某个字符串，并**执行其中的的 JS 代码**。而这个函数也出现在了oo0O0()里面，
```js
eval(atob(window['b'])[J('0x0', ']dQW')](J('0x1', 'GTu!'), '\x27' + mw + '\x27'));
```
eval()里面还调用了一个函数atob()，atob() 方法用于解码使用 base-64 编码的字符串。我们一步步来，先看`atob(window['b']`是什么，复制到控制台运行后
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210124230811.png" style="zoom: 80%;" />
将代码复制格式化之后得到的是MD5加密算法，非常长，这里截取一部分
![](https://gitee.com/lonercci/picbed/raw/master/img/20210124231335.png)
通过最后一行代码，我们发现 `window.f` 出现了，并且通过调用hex_md5()来实现赋值。  
但是又出现了一个新的问题，`mwqqppz`是什么？

### 寻找mwqqppz
再来看一下刚刚的eval代码片段
```js
eval(atob(window['b'])[J('0x0', ']dQW')](J('0x1', 'GTu!'), '\x27' + mw + '\x27'));
```
可以看到eval函数里面除了atob(window['b'],还有
+ `J('0x0', ']dQW')`
+ `J('0x1', 'GTu!')`
+ `'\x27' + mw + '\x27'`
我们再到调试工具的Console运行一下上面列出来的代码
![](https://gitee.com/lonercci/picbed/raw/master/img/20210124232140.png)
这是因为oo0O0()函数里面还有一些代码段没有执行
我们先把oo0O0()函数里面eval函数上面的所有代码复制,然后去Console中运行一下
```js
    var U = ['W5r5W6VdIHZcT8kU', 'WQ8CWRaxWQirAW=='];
    var J = function (o, E) {
        o = o - 0x0;
        var N = U[o];
        if (J['bSSGte'] === undefined) {
            var Y = function (w) {
                var m = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=',
                T = String(w)['replace'](/=+$/, '');
                var A = '';
                for (var C = 0x0, b, W, l = 0x0; W = T['charAt'](l++); ~W && (b = C % 0x4 ? b * 0x40 + W : W, C++ % 0x4) ? A += String['fromCharCode'](0xff & b >> (-0x2 * C & 0x6)) : 0x0) {
                    W = m['indexOf'](W)
                }
                return A
            };
            var t = function (w, m) {
                var T = [],
                A = 0x0,
                C,
                b = '',
                W = '';
                w = Y(w);
                for (var R = 0x0, v = w['length']; R < v; R++) {
                    W += '%' + ('00' + w['charCodeAt'](R)['toString'](0x10))['slice'](-0x2)
                }
                w = decodeURIComponent(W);
                var l;
                for (l = 0x0; l < 0x100; l++) {
                    T[l] = l
                }
                for (l = 0x0; l < 0x100; l++) {
                    A = (A + T[l] + m['charCodeAt'](l % m['length'])) % 0x100,
                    C = T[l],
                    T[l] = T[A],
                    T[A] = C
                }
                l = 0x0,
                A = 0x0;
                for (var L = 0x0; L < w['length']; L++) {
                    l = (l + 0x1) % 0x100,
                    A = (A + T[l]) % 0x100,
                    C = T[l],
                    T[l] = T[A],
                    T[A] = C,
                    b += String['fromCharCode'](w['charCodeAt'](L) ^ T[(T[l] + T[A]) % 0x100])
                }
                return b
            };
            J['luAabU'] = t,
            J['qlVPZg'] = {},
            J['bSSGte'] = !![]
        }
        var H = J['qlVPZg'][o];
        return H === undefined ? (J['TUDBIJ'] === undefined && (J['TUDBIJ'] = !![]), N = J['luAabU'](N, E), J['qlVPZg'][o] = N) : N = H,
        N
    };
```
然后依次运行

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210124232743.png" style="zoom: 67%;" />

此时 mwqqppz 出现了

### 翻译eval()函数
知道eval()函数里面的奇怪符号的意义后,我们就可以翻译一下eval()代码片段
```js
eval(atob(window['b'])[J('0x0', ']dQW')](J('0x1', 'GTu!'), '\x27' + mw + '\x27'));\\原来的

eval(atob(window['b'])["replace"]("mwqqppz",'mw'));\\翻译后的
```
使用Python重写：
```python
eval(atob(window['b']).replace("mwqqppz",'mw'));
```

而mw是什么？是oo0O0()函数的一个形参，回顾前面的代码
```js
var timestamp = Date.parse(new Date()) + 100000000;
var m = oo0O0(timestamp.toString()) + window.f;
function oo0O0(mw) {
...
}
```
可见，其传入的就是一个字符串时间戳

### 思路整理+验证
很清晰的我们可以得出
```js
var timestamp = Date.parse(new Date()) + 100000000;
var m = oo0O0(timestamp.toString()) + window.f;
var m = window.f;
var m = hex_md5(mwqqppz);
var m = hex_md5(timestamp);
```
改写一下js文件，直接把时间戳带入，加载代码，对比相同，成功

## JS加密部分代码
```js
var hexcase = 0;
var b64pad = "";
var chrsz = 16;

function hex_md5(a) {
    return binl2hex(core_md5(str2binl(a), a.length * chrsz))
}

function b64_md5(a) {
    return binl2b64(core_md5(str2binl(a), a.length * chrsz))
}

function str_md5(a) {
    return binl2str(core_md5(str2binl(a), a.length * chrsz))
}

function hex_hmac_md5(a, b) {
    return binl2hex(core_hmac_md5(a, b))
}

function b64_hmac_md5(a, b) {
    return binl2b64(core_hmac_md5(a, b))
}

function str_hmac_md5(a, b) {
    return binl2str(core_hmac_md5(a, b))
}

function md5_vm_test() {
    return hex_md5("abc") == "900150983cd24fb0d6963f7d28e17f72"
}

function core_md5(p, k) {
    p[k >> 5] |= 128 << ((k) % 32);
    p[(((k + 64) >>> 9) << 4) + 14] = k;
    var o = 1732584193;
    var n = -271733879;
    var m = -1732584194;
    var l = 271733878;
    for (var g = 0; g < p.length; g += 16) {
        var j = o;
        var h = n;
        var f = m;
        var e = l;
        o = md5_ff(o, n, m, l, p[g + 0], 7, -680976936);
        l = md5_ff(l, o, n, m, p[g + 1], 12, -389564586);
        m = md5_ff(m, l, o, n, p[g + 2], 17, 606105819);
        n = md5_ff(n, m, l, o, p[g + 3], 22, -1044525330);
        o = md5_ff(o, n, m, l, p[g + 4], 7, -176418897);
        l = md5_ff(l, o, n, m, p[g + 5], 12, 1200080426);
        m = md5_ff(m, l, o, n, p[g + 6], 17, -1473231341);
        n = md5_ff(n, m, l, o, p[g + 7], 22, -45705983);
        o = md5_ff(o, n, m, l, p[g + 8], 7, 1770035416);
        l = md5_ff(l, o, n, m, p[g + 9], 12, -1958414417);
        m = md5_ff(m, l, o, n, p[g + 10], 17, -42063);
        n = md5_ff(n, m, l, o, p[g + 11], 22, -1990404162);
        o = md5_ff(o, n, m, l, p[g + 12], 7, 1804660682);
        l = md5_ff(l, o, n, m, p[g + 13], 12, -40341101);
        m = md5_ff(m, l, o, n, p[g + 14], 17, -1502002290);
        n = md5_ff(n, m, l, o, p[g + 15], 22, 1236535329);
        o = md5_gg(o, n, m, l, p[g + 1], 5, -165796510);
        l = md5_gg(l, o, n, m, p[g + 6], 9, -1069501632);
        m = md5_gg(m, l, o, n, p[g + 11], 14, 643717713);
        n = md5_gg(n, m, l, o, p[g + 0], 20, -373897302);
        o = md5_gg(o, n, m, l, p[g + 5], 5, -701558691);
        l = md5_gg(l, o, n, m, p[g + 10], 9, 38016083);
        m = md5_gg(m, l, o, n, p[g + 15], 14, -660478335);
        n = md5_gg(n, m, l, o, p[g + 4], 20, -405537848);
        o = md5_gg(o, n, m, l, p[g + 9], 5, 568446438);
        l = md5_gg(l, o, n, m, p[g + 14], 9, -1019803690);
        m = md5_gg(m, l, o, n, p[g + 3], 14, -187363961);
        n = md5_gg(n, m, l, o, p[g + 8], 20, 1163531501);
        o = md5_gg(o, n, m, l, p[g + 13], 5, -1444681467);
        l = md5_gg(l, o, n, m, p[g + 2], 9, -51403784);
        m = md5_gg(m, l, o, n, p[g + 7], 14, 1735328473);
        n = md5_gg(n, m, l, o, p[g + 12], 20, -1921207734);
        o = md5_hh(o, n, m, l, p[g + 5], 4, -378558);
        l = md5_hh(l, o, n, m, p[g + 8], 11, -2022574463);
        m = md5_hh(m, l, o, n, p[g + 11], 16, 1839030562);
        n = md5_hh(n, m, l, o, p[g + 14], 23, -35309556);
        o = md5_hh(o, n, m, l, p[g + 1], 4, -1530992060);
        l = md5_hh(l, o, n, m, p[g + 4], 11, 1272893353);
        m = md5_hh(m, l, o, n, p[g + 7], 16, -155497632);
        n = md5_hh(n, m, l, o, p[g + 10], 23, -1094730640);
        o = md5_hh(o, n, m, l, p[g + 13], 4, 681279174);
        l = md5_hh(l, o, n, m, p[g + 0], 11, -358537222);
        m = md5_hh(m, l, o, n, p[g + 3], 16, -722881979);
        n = md5_hh(n, m, l, o, p[g + 6], 23, 76029189);
        o = md5_hh(o, n, m, l, p[g + 9], 4, -640364487);
        l = md5_hh(l, o, n, m, p[g + 12], 11, -421815835);
        m = md5_hh(m, l, o, n, p[g + 15], 16, 530742520);
        n = md5_hh(n, m, l, o, p[g + 2], 23, -995338651);
        o = md5_ii(o, n, m, l, p[g + 0], 6, -198630844);
        l = md5_ii(l, o, n, m, p[g + 7], 10, 11261161415);
        m = md5_ii(m, l, o, n, p[g + 14], 15, -1416354905);
        n = md5_ii(n, m, l, o, p[g + 5], 21, -57434055);
        o = md5_ii(o, n, m, l, p[g + 12], 6, 1700485571);
        l = md5_ii(l, o, n, m, p[g + 3], 10, -1894446606);
        m = md5_ii(m, l, o, n, p[g + 10], 15, -1051523);
        n = md5_ii(n, m, l, o, p[g + 1], 21, -2054922799);
        o = md5_ii(o, n, m, l, p[g + 8], 6, 1873313359);
        l = md5_ii(l, o, n, m, p[g + 15], 10, -30611744);
        m = md5_ii(m, l, o, n, p[g + 6], 15, -1560198380);
        n = md5_ii(n, m, l, o, p[g + 13], 21, 1309151649);
        o = md5_ii(o, n, m, l, p[g + 4], 6, -145523070);
        l = md5_ii(l, o, n, m, p[g + 11], 10, -1120210379);
        m = md5_ii(m, l, o, n, p[g + 2], 15, 718787259);
        n = md5_ii(n, m, l, o, p[g + 9], 21, -343485551);
        o = safe_add(o, j);
        n = safe_add(n, h);
        m = safe_add(m, f);
        l = safe_add(l, e)
    }
    return Array(o, n, m, l)
}

function md5_cmn(h, e, d, c, g, f) {
    return safe_add(bit_rol(safe_add(safe_add(e, h), safe_add(c, f)), g), d)
}

function md5_ff(g, f, k, j, e, i, h) {
    return md5_cmn((f & k) | ((~f) & j), g, f, e, i, h)
}

function md5_gg(g, f, k, j, e, i, h) {
    return md5_cmn((f & j) | (k & (~j)), g, f, e, i, h)
}

function md5_hh(g, f, k, j, e, i, h) {
    return md5_cmn(f ^ k ^ j, g, f, e, i, h)
}

function md5_ii(g, f, k, j, e, i, h) {
    return md5_cmn(k ^ (f | (~j)), g, f, e, i, h)
}

function core_hmac_md5(c, f) {
    var e = str2binl(c);
    if (e.length > 16) {
        e = core_md5(e, c.length * chrsz)
    }
    var a = Array(16), d = Array(16);
    for (var b = 0; b < 16; b++) {
        a[b] = e[b] ^ 909522486;
        d[b] = e[b] ^ 1549556828
    }
    var g = core_md5(a.concat(str2binl(f)), 512 + f.length * chrsz);
    return core_md5(d.concat(g), 512 + 128)
}

function safe_add(a, d) {
    var c = (a & 65535) + (d & 65535);
    var b = (a >> 16) + (d >> 16) + (c >> 16);
    return (b << 16) | (c & 65535)
}

function bit_rol(a, b) {
    return (a << b) | (a >>> (32 - b))
}

function str2binl(d) {
    var c = Array();
    var a = (1 << chrsz) - 1;
    for (var b = 0; b < d.length * chrsz; b += chrsz) {
        c[b >> 5] |= (d.charCodeAt(b / chrsz) & a) << (b % 32)
    }
    return c
}

function binl2str(c) {
    var d = "";
    var a = (1 << chrsz) - 1;
    for (var b = 0; b < c.length * 32; b += chrsz) {
        d += String.fromCharCode((c[b >> 5] >>> (b % 32)) & a)
    }
    return d
}

function binl2hex(c) {
    var b = hexcase ? "0123456789ABCDEF" : "0123456789abcdef";
    var d = "";
    for (var a = 0; a < c.length * 4; a++) {
        d += b.charAt((c[a >> 2] >> ((a % 4) * 8 + 4)) & 15) + b.charAt((c[a >> 2] >> ((a % 4) * 8)) & 15)
    }
    return d
}

function binl2b64(d) {
    var c = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    var f = "";
    for (var b = 0; b < d.length * 4; b += 3) {
        var e = (((d[b >> 2] >> 8 * (b % 4)) & 255) << 16) | (((d[b + 1 >> 2] >> 8 * ((b + 1) % 4)) & 255) << 8) | ((d[b + 2 >> 2] >> 8 * ((b + 2) % 4)) & 255);
        for (var a = 0; a < 4; a++) {
            if (b * 8 + a * 6 > d.length * 32) {
                f += b64pad
            } else {
                f += c.charAt((e >> 6 * (3 - a)) & 63)
            }
        }
    }
    return f
};
function yanzheng(){
	 var timestamp = Date.parse(new Date()) +100000000 ;
    timestamp +='';
    f = hex_md5(timestamp);
    m = f + '丨' + timestamp / 1000
    return m;
}

```
## 爬虫代码
```python
import requests
import execjs
import time


def get_psd_m():
    #导入js文件
    with open(r'get_m.js', encoding='utf-8', mode='r') as f:
        JsData = f.read()
    # 加载js文件,使用call()函数执行,传入需要执行函数即可获取返回值
    psd = execjs.compile(JsData).call('yanzheng')
    psd = psd.replace('丨', '%E4%B8%A8')
    return psd

def get_data(page,psd):
    url = "http://match.yuanrenxue.com/api/match/1?page={}&m={}".format(page, psd)
    headers = {
        'User-Agent':'yuanrenxue.project',
        'Referer': 'http://match.yuanrenxue.com/match/1'
    }
    r = requests.get(url,headers = headers)
    return r.json()


if __name__ == '__main__':
    # 初始化每页数据的总和，数据的个数
    sum_list = 0
    index = 0

    for page in range(1,6):
        info = get_data(page,get_psd_m())
        price_list = [i["value"] for i in info["data"]]
        print("这是第{}页的价格列表\n{}".format(page,price_list))
        # 自加操作
        sum_list += sum(price_list)
        index += len(price_list)
        # 防止爬取太快
        time.sleep(1)

    avg = sum_list / index
    print("平均价格为{}".format(avg))
```
运行结果：
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210126000219.png" style="zoom:80%;" />
## 题目总结
   这是整个比赛题中的第一题，题目难度标注为简单。但是对我这个菜鸟来说，一上来就被卡在了前几步。阅读几位大佬的博客之后，对这道题有了一个大致的把握。在JS的逆向方面，一步一步的去分析代码，思考通过什么关键词去定位函数，注意是否在调用时发生了变化。
   本题名字叫JS混淆，我也不知道具体混淆的意思。但是在对JS的分析当中，似乎在`eval()`部分用到了很简单的混淆，把某些函数名和参数使用它自己的js加密，成为一个个看似毫无里头的字符串，我们要做的就是把它翻译回去。
   最后，非常感谢 [`Java_S`](https://syjun.vip/archives/278.html) 大佬的博客，参考了很多内容。他的分析思路非常清晰，步骤详细，代码干净简洁，看着太舒了，，，，向大佬学习。
