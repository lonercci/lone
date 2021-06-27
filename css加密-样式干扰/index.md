# css加密-样式干扰

## 题目
> 本次题目涉及到了一些前端的知识，包括HTML，CSS，JQuery，Ajax。

![](https://gitee.com/lonercci/picbed/raw/master/img/20210129185551.png)

> 采集这5页的全部数字，计算加和并提交结果
题目链接：http://match.yuanrenxue.com/match/4

## 接口数据分析
页面上的数字感觉跟正常的不一样的，我们随意在页面上点击一个数字，发现每个数字都是由图片组成的。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129213956.png" style="zoom:50%;" />

果然在all里面有0-9的数字图片，每张图片都是base64格式。我们打开xhr看看，这里的数据有点奇怪，好像没有跟上面几道题一样有value值，info字段里面是一段很长的东西。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129213426.png" style="zoom:50%;" />



我们来仔细看看info部分，发现是一段html代码。复制粘贴下来格式化一下，

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129224629.png" style="zoom:50%;" />

我们有如下思考：

1. src对应的是base64编码的图片
2. class属性的值很像是MD5加密之后的东西
3. style中的left，我们猜测可能是偏移什么的
4. 根据网页中的数据，每页有只有10个四位数。每个四位数似乎对应着td标签的图片，即四个src的值，那么每一部分的代码行数应该是一样的，为什么会出现有多余四张图片的情况？

我们带着这些问题来看看网页代码

## 分析网页
查看一下网页源代码
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129215147.png" style="zoom:50%;" />

分析一下，我们可以得到如下信息：

1. 每一个`<td>`标签代表一个四位数
2. `<td>`标签中包含`<img>`标签，但是每个`<td>`中的`<img>`数量可能还不一样的。这是因为标签中,有些是有`display:none`样式的,这个样式设置了不显示此标签,也就是不显示这个图片。我们观察发现，排除这些不显示的图片,其实`<td>`标签中都有且只有四张图片正常显示,只不过数字不一样罢了。如图，网页中只有四位数`6081`，按理说源码中也是四张图片，但是我们在源码中却可以看到有六张图片，每张图片合起来却是是`360158`。  
3. 每个标签都设置了`left`这个样式，而`left`的作用可以改变图片在页面中显示的位置（**把图像的左边缘设置在其包含元素左边缘向右 xx 像素的位置**）。这是什么意思呢?如图，我们排除隐藏的图片，也不管图片的偏移，按照正常加载顺序四位数应该是`6018`，但是第三个数字css属性是`left:11.5px`，向右移动了11.5px,第四位数字css属性是`left:-11.5px`，向左移动了11.5px，即互换了位置。于是加载以后网页上显示的是`6081`。

这样，似乎只解答了一部分问题。我们知道接口里的数据个数大于四位是有问题的，但是没关系，网页中会把某些数据不显示，我们看到的还是四位数。那么新的问题来了，它是怎么做到的，按照什么规则把某些图片不显示的。


## 寻找算法(js)
> 我们可以猜测，网页里肯定是通过了某些算法，进行了删选。于是现在的目标就是如何定位，寻找到这部分的js。

这里我们有一个突破口，`display:none`，因为在`<td>`标签内存在含义`display:none`样式的`<img>`标签,而设置`display:none`这个样式大概率就会出现我们寻找的js里面。

我们全局搜索一下` 'display','none' ` 。
**为什么是搜索` 'display','none' `而不是`display:none`呢?**

因为【display:none】是结果,【 'display','none' 】是过程，执行某种算法就是一个过程。

在JQuery中有一个可以设置标签样式的函数--->css()

第一个参数就是需要设置的样式,第二个参数就是设置样式的值,也就是这样css('display','none')

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129231454.png" style="zoom:50%;" />

可以看到我们要找的内容,出现在一段`<script>`标签中,我们将其中的JS代码扣下来继续分析

## 分析JS代码
因为有些代码片段是有报错才执行,所以我们可以将扣下来的代码精简一下
```js
window.url = '/api/match/4'; // 设置一个url链接
request = function () {
    var list = {
        "page": window.page,// 设置页码
    };
    $.ajax({                // ajax请求
        url: window.url,    // 请求的网址,也就是上面设置的url链接
        dataType: "json",   // 预计返回信息类型:json
        async: false,       // 是否开启异步请求:否
        data: list,         // 发送给服务器的信息,也就是上面设置的页面
        type: "GET",        // 请求方式,GET
        beforeSend: function (request) {},　// 可以不管
        // 请求网址成功执行下列代码
        success: function (data) {
            datas = data.info;
            $('.number').text('').append(datas);
            var j_key = '.' + hex_md5(btoa(data.key + data.value).replace(/=/g, ''));
            $(j_key).css('display', 'none');
            $('.img_number').removeClass().addClass('img_number')
        }
    })
};
request()
```
通过上面的代码,我们可以知道网页通过ajax的方式请求了[http://match.yuanrenxue.com/api/match/4] 跟第一步的猜测一样

再回到ajax的代码中,看看请求网址成功执行的代码
```js
// 首先我们需要搞清楚的是,函数里面传入的参数**data**,跟上面的data含义是不一样的
// 这里的data值的是请求成功后返回的内容
success: function (data) {
    // data.info就是获取返回内容中info的部分,也就是<td>标签里面的内容
    datas = data.info;
    // 通过JQuery的选择器,选择标签中有number的元素,并将<td>标签的内容添加进去
    $('.number').text('').append(datas);
    // 通过获取data中key和value中的值,并替换一些字符,最后经过md5算法加密,得到一串密文
    // 在密文前面加上一个【.】,并赋值给变量j_key
    var j_key = '.' + hex_md5(btoa(data.key + data.value).replace(/=/g, ''));
    // 因为密文前面加了一个【.】,所以可以通过选择器,选择有密文这个属性的标签,并将其样式设置成【display:none】
    $(j_key).css('display', 'none');
    // 选择含有img_number属性的元素,先删除所以元素,最后添加img_number元素
    // 为什么要这么做,是为了删除属性中带着的密文
    $('.img_number').removeClass().addClass('img_number')
}
```
我们再拿一下第一步的图
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129224629.png" style="zoom:50%;" />
可以看到`<img>`标签中的属性部分,是有密文部分的,所以就解释了为什么会有这段代码，
```js
// 选择含有img_number属性的元素,先删除所以元素,最后添加img_number元素
// 为什么要这么做,是为了删除属性中带着的密文
$('.img_number').removeClass().addClass('img_number')
```
还有就是,style里面的内容,也就是【left】中的值,决定了图片所在的位置。

到此，已经解决了当时的所有问题。

我们整理一下css加密的思路
1. 通过ajax请求的方式请求带有数据的链接`/api/match/4`
2. 请求成功后,将info中的`<td>`标签添加到HTML代码中
3. 通过md5算法的一系列操作,得到一串密文
4. 匹配有密文属性的`<img>`标签,将其设置为`display:none`不显示
5. 接着,将属性中的密文删除(因为已经不需要带加密的了)
6. 最后,通过附加的`left`样式,页面中就呈现出了需要我们求和的数字

## python爬虫
理清之后发现思路并不复杂。但是如何把上述的过程使用python代码来一步步的还原，让我有点犯难了。我们来思考一下如何用代码复现：
在接口里，我们可以拿到一段标签。里面包括base64编码之后的图片值、class属性里面有md5加密后的密文、以及css的偏移量。
1. 想要爬取数字，首先要做一张映射表。把base64值和代表的数字进行关联，做到一一对应。
2. 我们知道，接口里面的可能会出现大于四张base64的图片的情况。因为接口里的数据是没有display属性的。那么我们就需要对所有的base64值进行筛选。如何筛选？模拟网页中的js，比对密文，把不显示的去掉。这时，我们每个四位数正好有四个base64的值对应。
3. 为了方便表示，我们通过数字与base64的映射关系，拿到每页的每个四位数表示的数字。
4. 但是此时的四位数是按照默认顺序排列的，我们要做的就是根据style里面的偏移量进行计算，还原得到正确的排序方式。
5. 拿到所有的数字，进行加和
当然上述过程中肯定要通过正则表达式把需要的内容从接口数据里拿到，并进行匹配


```python

import re
import base64
import hashlib
import requests

image_dict = {
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAdCAYAAACqhkzFAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAMTSURBVEhLrZY/TBNRHMe/bY82LeGA0Dp4NTGYqO1gdKEMwmQcGh38M5CQlGDSwWiMAwwYg2iCJsZJQ9CJ0ETsQMJgwuagLIUFXIAFXNoYcqDloLRXrq3v7v3aXktbSPCTXO77fZRvf/feu9+r5dz5iwX8R+oESigMPka6/wb2vSJUBx+1QoVD3kTz7DScb+f4YBVHA6U+ZKLPsON1IE9DtbDLP9AxEIawRgOEle4cKYSD+ZeQq8IcKquMXTbyOllPL7aiEWgSDRCmQAnahyHsiGQZruUvkIKXcObyFeM62zcOd1ylvwJ5MYDkuxA5Tjlw8A2S12iyGM7YODrujsFqfqTFCJw9L+CWyTPS3QPIBMgwKFBC9n4AaW4AdQVtQxEy1czBObkIJznAi9TDO6SLgVIYqt9QBq5YFEKCTC2momg2VXngD5bmnAc+uIqUIXQUOL/X3hJl5mHfUEgzPJ04pMUxAnN+CYeG1UlAmCLZANtqAk2kARG5m1zxQI9paWUZAsmGsArLnxOh0ZSxwCBypjwo2zQPxzCTgJ2kTtbbZ9zZ/7pRKO8WOOWqrX9iePyJiqnNGgTTuhRhgT5o5kc+JaeosDYssHbpx+OtXEziSIWqx0fqOFpg7ns2Zd24s8B1WE0V5h2mJW/ELTc0kjo2Zcm4s8ClikCIlR+si59tZpL66yosc2U8clM8bhgDkfVFUzuqR569rqXOqG7CPsOlEWiJJSraUaa/i3Q9upC94CHNCtr4WXoN+aJMzcFlakd73eGG5wnuhZHykmZ1tnx9TboYqDfNWLxoWDvqhfK+h0w1Pcg87cUBOavMmu1HMoxShuXJNNpNi7N3ewLJTyEUzIdQIIT0wgQ7EcmzxWifZMcEOZ3KY3Qwgp3RQOnbOfqJx5XGtlSOS4aK1tlHEIcXyHNsrW0dY6SBFX0ufShc70S21OwE5AR+lb+ZVfb5OVpGviGTTkHeSmBfScJqtdX55SAFoY0OsMXxISM6SvvNrshwrS7A9WoENupyv+O/oGm831sslno/RU6OsvsHu3+3yQH/AOyW6SvqnweCAAAAAElFTkSuQmCC': 0,
    'iVBORw0KGgoAAAANSUhEUgAAABUAAAAcCAYAAACOGPReAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAD0SURBVEhLY5RVUPvPQGXABKUJAA2GvzP3Mry6f5PhzfI4qBhuQNjQqD6Gzxc3Mjxzk2H4CRUiBHAYKs3wP7Gd4evhSwzPWr0ZPvBBhYkEqIaaBzL8nrmS4f3FfQzP6oIY3smwM/yFSpECkAyNY/g2q4PhhZsBwxegy/5BRZlfP2HgIdbfUIAnTD8xCGyuZ5AyW8jATqmhzD9fMwjumsUgbWPKwJu3AipKGkAydC8DZ24sg5SGDQNPei8D01OoMBkAydCnDIyHTkHZlAE8YUo+GDWU+mAEGfq46RCUhQAUGypbZwdlIcBoRFEfjBpKfUADQxkYAKYHOb9g+7HMAAAAAElFTkSuQmCC': 1,
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAdCAYAAACqhkzFAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAKCSURBVEhLrZZLaBNRFIb/PDp5SFMxjqVOAjYLNUGUbhI3cdEGwaAgbmopWFIQBEUwFhQVldIqCuqilLoymEVVlCqWuNdVXRk31Y1uTKAQMRI0ycQk453JaWaSJjYPPxjmnJPwcefOuSfROXftlvAfaSAUIIXOITfuR9bBQzQBZfqEEzPgVuPY8uIeuMXPVK2lVnjoCrKzJ5F2mKqSZpi/xGCfCEOfpAKhCkPz+HEpgN9sRVpMoqjcJZMJBSVS0Wfeoz94CkaNVE93wOOqygyZr9i+cBnOwT3YsXe/cvUPDsMxHcPWTOU7MmWbD+m5i5RVUIUKGfQt38DOA0dgufuSauskoYuE0RuchV0jzQ8FIAqUMFShmIB9+jhs559SoQnJKKwLcZgpBVwoTFLIUIXXTsMaqdvhZjz8qBECRYeXog2P3CoJts8UMko2F0UdC90o2ihkcAl1mzoTnnGh0kwyGRhXKWR0IBRQOOZGnjL57RsjFDLaF87MI+1Ru9+ysgSOYpn2hKEovo+71RMjfoJtKkpJhRaFAsozr5C67kOOKvLe2e+cBVfXaZsLhSD+LD3DGluZum8p5RA06tt/C4/exK83D7A2xKNEJT07UfzV0aaHoImQHnFuDGlNv8kja2BkBObF5ieqgdDf8BG3PboAPrBx/tVTN7H9KMTuI+WxVQesIRUHPzWKnndU2ATNCgUUn9TKrB8eY8DbukxGFYZu4+dBVWZZYXPvxC3oKG8VEnqRn1R7rCfBfi/Gahu2VSpC3wRyDiViiOhdDre9snUqwsOCZnqkYHhLYSfIb9n5+psESerq6nvOPMylrLDEa7q3C7L7hrVt0z1Fu9Dor0g3AH8BJlTqZkAngxQAAAAASUVORK5CYII=': 2,
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAdCAYAAACqhkzFAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAALaSURBVEhLrZVPSBRRHMe/O+rOuuaUyRoyBlGQfw5lh9xTekg8qIdMKSsQFKRAD0WUROhBCIlSBD1YQsJKQSB1iRa8VRf1Up4UQiNcRVw1nXVdx3V3euP+ZnZmR9EVP/CY3/vx5sPMe+/3nu3suYsKjhGrUCxB5GErNssKERIEyDzlIYOX/MgY+4r0zm5w85ROwCRU2j1Yu+fGhi7ZG072Iau3Bc6BacrE4ejJ6EKgySyzy+yrdptpIKJ8HlbahrHZKFImjnEcIeHU6FuIlfk4U3AJObstH2L9C7hmJKTQKEBg0h7sUE/DJLT5x5FbeRWZ99kcTVFSY9wDR/kN5Pxkn6vBFyPUTjFhEPYjs6YBqYkiE/NIfTeODOqpyEUNFMUwCNmy7bNyJr5M787pfuwxhwfhh+1YhWIxtgWKGXafh6IYSQuVjssIUgxMwdFLIZGc8IEHKxV5iFL3xGg37AnzfnAtu68jUlYFueIa1i4IiFDaOfYa2XcGqRfHKnzlxVzdeepYSZVmkfXyERwfrGWnktQvc7If6T62yrxhVRKwCvX6jTet3KK8C4EiN5Y6hrEw6YV811rLhzwPRSi19dhqqmFCFzvINPzI7rwN51B8ZQ4pNNA4iNW2UgTpVOL835Fb0qz/atL7EEPNyPrm01+MutwIPaUOI3khw9Y5aTggeISu1FN8RCHm5YQX7fQ8qlDk9WpR4eRlinQhW37rDtgXpc1Yz2xrsYtLg4StCHz+iHD1IazVPQgY6hnSLzgGKGbov6y4irHY58W/T13YKS2grIHCGoTfeLHcV4V1w9V6+v0zwz2j78MuSH9uYp2SKhwbnKbvYB5hJjHOmyo7OdIC4ckP6segL5wAb7rR1Jd5dslrzSxLYQeE6/ktXbYVCmJp0YfghpRQKbWPsVVXjs0iEWGBxzalVXhZQtrMFJwj/eCHJigbY2FuFpFI7EJNvvT2wPf3NxRF1QD/AbAv8WdRHzjKAAAAAElFTkSuQmCC': 3,
    'iVBORw0KGgoAAAANSUhEUgAAABUAAAAbCAYAAACTHcTmAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAHHSURBVEhLrZW9LwNhHMe/WrT10mrKVE2EROIfUAuTTQxMLISBQSxeIjQSTZRIsYgYhYVRIrWzaCxiMjFVoqjWaatPaR9Pe09bbfT6XHKf5Mn9vne9T+55fs/1qhxtnRRqWD1DcLILSR5Nfg+aR495ktHxoyBDiA8WhOVQJaXeKUgtPCggLrXPs6dsxzePSghLU9sjiBh4qICYtG8fUo8Z6UxNXlEvZc+WRUDaC+LpR5SnhssLGHldjopS6l1BpFWuda9XsEwH5KCAstS5iU/WHHkLSbAerAmtl8JvupH0DOCDN8d4e4q6wyc5VKCslHrdCHdwI7mHZXZHrgX4X9r3d9oElvMN1Io9ZJZ/pL1IeIbz06598KFx8UYOgpRI7fg52UWIdzszbev4Mqp4FKVYOrGJSG6Ts27btmZUTTtHQcrWUVpy4otHk39PuNulyFL7GOJ7hXWsCfhgK/mPVAOTstfwaAEhMz9DHmF1zalexyIcbj81UUqh0TBdr9OS7muDDoRAz4ZBxajmN2dggqJrepKE+g8fWFPvXPkeaPDhE0MzaeIrhpfnAGJRSTvp+1sQJBFHOBTUTppOp7LHzM7STGpusvEK+AUL4d3X/AgqvQAAAABJRU5ErkJggg==': 4,
    'iVBORw0KGgoAAAANSUhEUgAAABUAAAAcCAYAAACOGPReAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAIxSURBVEhLrZU/aBNRHMe/bWKexnoiaSg0FSSCaAfFwT+L7dKloUs7OqRVcHBwcZOCiCAOhk4dxAwdHFRwEEoRSjvoZNqhdDE4tC7aIYmWvOiV1941vnf51dy7S0ou6Qfu+P5+B9/7/e7e+72us+cuVHHE+EzF0jcUzlPQCjyH/itphChUdKub+bcCXv4tVQr7hsp0hmNqWXsob5ek6kWVqUxneNpPw1yfxi+nWo7Y02uIzjkPAuFUWucSrKNqvzEcoUWSAdFNbyewSxJK/SQZEN1U+0kCXaSCopvGGfZJgsv2SQbFY2rI+jqn+Y/ipbYr1dap/WYVWzfra0q98f/nkDDBcWwjj+j7WbC5Fcr6aV6pxG2oEMzAn8EbKDx+ja31jxBytTRCM7WNepVhIWRl6qKEB9tIovDsA8w7fuPWR9/QPezenQAfTmKHUg5iE32To4jkKJYc2r7G5ywiU6PoTWUQK7rKZ0lU7o9TUKN10wPyWUQfLOA0hQrz6jhs0orgporcI0TXXNUacVgkFe2ZSsI/iqT8tG162DBv23RvYICURG4K3xkVmKEZmIOkJWqXhUkrgpsm0tjJjKBCoRqRp+afkK7hMr2I6tgt0g1IXIf94h22l6dRkiPygMjXtzj5kgLCtaOeg3+fQNnR/u1pMaatRUVkYwHxyYfo9pwQTdpncnjol24o0PMpg74Rv6HCZbqC41/y6OG1QeKdpSGZO1HcRGz+FRKpyzgzlaUnXoB/3J2gmVZucHAAAAAASUVORK5CYII=': 5,
    'iVBORw0KGgoAAAANSUhEUgAAABUAAAAdCAYAAABFRCf7AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAANLSURBVEhLrZZNSFRRFMf/48e8UZtRktFkRsiEdCIKVwaikrlJbZG0MEGDwEWUIPaBQoiIZIJGG4mikoyyIBJKW4gbs4W66WOhtlAXzixq/HzGjG/8eN0378yb+5w3qdAPLnPOmXf/nHfvOfc+U+bR4zL+M/sTrW6F71Ih/CfskAQB2xQ2SyLMU5OwPr+HuEEPRfcSddXB96QeK04BOxQyZg7pWedhJi+GfiOp7sHqwC0s7RKMkyQIwaGfzGdmLFrUAbGlFOsC+ZBwaLwfjqoSZOSeQlpw5MBR1oQjw3NIoKdCGLz+Rfgn72PRTq7khr2tFpbX4TWLwJULTM+QY5Cp/PgGlkOCEJHauYegAieosEuUbUyxU1tDy9c3SOzdQ9AAvWh7ObeOblg7u8k+GJyoA4E8FzbJi5mdhGWCnAPCiVZhM5tMRuLsAFkHJyxakcu6hWy2QcL4pGpW3IR/6AsWZ35iYV4dnvkfWBp7C/+dMvWZ3SglFRzPpuR4WZYRHAtyWsFZ2fFhVha0mPEQFkbljALSoBHO1G7V1hMIQG54Cu+FY6zsVcyhTiI/hOQswq9PfdhyUIChiW7bbWQpOLDOBANMMknppLIcpIc6KasEzrYhJIv0KGPHlo/VrlrydBvFI2CDNXfyu+s4fLkVMdMUDuKBqbcRtvr3SA69BsN/phIByjaKKPvDPQLr7THyDPjcDOuwmxNwYaNBtaKKWse7YSI7GqbO70giWyHgVJdAE431covESipuah/t6fkGMzdNsruCv+FMveuIJzMavpMlEItryIskdGaERYc9XLnYsO0kk2Mr1YG1c3XkRZLgVXc0LDoxAYF7lY28yMm20ZfIbCkij5GfD4mrxFhRPQK5jepjR52XbEW0XCuRaOxcO40/ZLP1g+Wj2tqcKHMesZuRbKVEVl506O4eHRU9WCvWTnPEz47BMqjaOlFMNLJSErVgILsSiyMPsJ3Pp5yLnfY+LHeVclm6kdLWrJWgwR1VCGmkB7+z9V2u9H5wErv3uUZiiEh5xZK5G24UfaZBxiBcaUL6lIhYiigEFLFdgjHsUkztrNEJKvz7Y6K6A76rhfA5lS+TUB1KrHQ8SBztR8LDPpgMemR/nz0HAvgL80YzEyuMQpQAAAAASUVORK5CYII=': 6,
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAeCAYAAAAsEj5rAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAHhSURBVEhLrZY9LANhGMf/PprDVQkxyFXSVCKIwYStiZoqhoqEhTCQWCwiBBEDiUTMBpOJhKQDkw1L1YDFxoSlBi2tlurrrn2uH9xX2/sll/6f4X557n3e903LWhxtDCZSTr/mIXXY4thgjSHGwEp76o7amOkdmih8BHcMlDAULz4DW3htSlf8+RIaJn3Fd8i2ZxAiGeK3sK34UrE4oTCP6JAT31TWneyg8jmdixKyNQ/eOCqCF7AuBKgoRigs48NlR5LK+rP1PEnBwuRmfnf8Kn0rUZhQWEfEJU/if3cSBQmTmwN4o6zUnYRxobh2kb5sd7X+PcWXDQvZohthee3EfWedy042F4PCCXzmTLbGf4hKyn8xJtwYRthGGU/gd9OnQgkDwh7EXB2ZU2G5v0TVFRUK6At7xxCzUxbhb/YoKaMrTM724J2yNIxqha2Si47Qi1hndqtw9wHFYUS7+hF2jaeytnDKi2jGFwfv36GcT6JRQMg9ncqaF+zPwTVe+mi84uc2t4+qbhcZjQ49+GrN7BVYHu50ZRLqwt5BxLLLh6qHfUraqAtHnIhSFG8CcGfa05VRFSZahcxRQ/wZllPKOqgIPfi2yzeBSPAVFRT1UBF2I5GzflzwEWWU9TD5zxLwC1sVsHrJiVs0AAAAAElFTkSuQmCC': 7,
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAdCAYAAACqhkzFAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAANtSURBVEhLrZXfS1NhGMe/rumZbR4nOcO2IA1CvQi8UW8y8EJqZCR1IQiKwvRCuiqTfomtkkrqIpN+EFSGNaFECL0IbzQvnH/A6EavNiinJVubO/5a73nPs51zNqf04wPjfJ8X9uV5n/d93ifr8JFjcfxHMhueuYS1dieiFTZIgoAtvihBCAVhnptErvshDAG+qGMHwzJsvh7Cj5MO9vfMGCQ/Ckb6sP/2F1pRMNCXsGPz/Vt8TzEzSiwz/qMFxrbgwEr7I0Tb7LSioDe8OYSVGhHbFOYEZ1DcVIfisuMo4r86ONxTyEsai1jpGcK6xlNj6MRaQznWKTL4p1BU5YLRqy1UAFmvumDtGEN+wlQoR7SnigKd4Qls2EiyDVs/dSGLojRmrsIyF6QAiFQ0ktIaslokspMzMT4gmQFDMEyK1ZO+MqqhL4x9JFn1EK8m+Yeohl4vu2Ok4UCsWa1LOmw3FaWkAbNfvTqaGg4jd86fXAjX34JUS0EK8YGX+FlBARZhfjpJWmfIbnlnHwoX6PiEUiy9mEVkwIV4ubKEWheksVksXSilekvI/9APwcsDTnqn2JsQ81zDikPQFTsdCXmf+2Ht9FCsoMuQE/DAdHkQBeqt2BHz9CDy3XozmZQM7dh+Pozlen3rGViUzRa22COxSWsycj8fcLfA9E69/JoM5T4exTeNWe7CBA6x1rOXKK1XXCK33gSsdBvkfg7eHdX1s5phG8ustxprPJCL3QWxW/+SJLE7Ib25h6WjghKHvCh2tsDIEqUMqxBrT5ixa+3zZDaTCUxCaPXAmtiKWI1f1M+KYXUr1hxcMSRYpvtJ70KgH7k+tdLhGhe/FYphvV1zCHv3cQLjguYlsjn4gSmGooANLv4N+TAUw2AIVF5GIbbaSO7BlqOQFEMK8cdFMRxZhIkLGRGRBhfp3XAhVimSZkbBgMYw8AQmH1ecWGVH2qzQEo3kYfVUGKvqtpA35+EPsmLIDiLn/hQsSsBgs6J3HKHHLYin+p5n47XtLMLP3MleN7DZY+me51rXevE741huLkeM4gQ5bOIp40BgM5qLJAZpEYUdpyHMUKx8FLJunIPtutpaCdZZD8vDPtXM5J/BwUbVTGaHQS9Thu0rF9kUrELEJmKDGSW2J7DTzPbNwzIyiOyPX2lVJYPh3wL8BvLZG6cpuRANAAAAAElFTkSuQmCC': 8,
    'iVBORw0KGgoAAAANSUhEUgAAABQAAAAcCAYAAABh2p9gAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAMzSURBVEhLpZZPSBRhGMYfV9dpdV3UHDVXIw1KDUwJVy/VLTAVWugQRJqUpxAkFyuMIBMPWRGYiCGGC7J0KCRKL13Si9pBvKiH9NIYoavgqKvjujt9M/OuM+uustUPvpn3edl9eL/3+7Mbl3fqjIxoFDnhb76Bzcoi7No47FI6QRJxbHYKKQOdSPi8RFmdKIZ2BDt6sHazCNuUiY4E67cXSLvtJq1hojdhx57nPX4fMDOxL3OSMiihwmHzchu8njrSGmGGclc/Vit5BEibxEXw7bdgzy9BZqEyziLP5UH6Cn2AsV3ZBF+DnVTYlBuxNe/CGqcpSHPIdl6DeY60EXsdfCNtWLVp0rQyhhOORrU6vcKOamyEzNgUU4fuRTdTWHIj6fUkLCSDfCl2GrR433CvuAB+ipXqLM8iVzCMd29gFSiGDb4rWi/J0IE9fr88QFhEAoWHMwWzIFLMeplbAaV3ZFiIIPXjb4gXvBQx+Ax1MfUeGuEM1cYKZzMauhFv2Arg2X6k8CgCuRkUKaQgWGOo0CwYHLlS+LocJA7B3oKdMmOfOMi8wdDUy84nxQrrtU8h1egbNpyLkAbr9T1rQO/h5EtYJ0Q9wRVguXsYG30tCFwq1HJ2B4KtPdiY6cfyac3NrD4VJMQpk1ROij7uyJk/dmTIcgxjXT7+aUK27OsFOZN56BWqjIOrf4isWRHxlImOiLSh+0iaJqnCKmTPA4aMpREkVpcjx/UWGdOLSBZDNw0booC0iY/IuVoO6+NxoJjXbyVRK+LwCzYGAp7v+FWprbR5dhDZ1Z1RKowZB/y5+rbhhK/qO2bD/FMncb7kHClGRT07vxRDgGVgSo1iNrTZUnChrIQUm26zA5sUm4QZcJNa/G89bHDD+6SCFkRCem8Vkp9r1114hfbDToaBmlcQH4TMgMSFL0giM4Vww2Z2AqaGsd3qpIQB5We1bxTe7mqsh46ctIjU9kfq/gsRPuWuUfy8XkCCrRzbexTBz0yCpBRMzCzDdRfcgd/mIxdFYveiNsLNOGEMWc6qCDOFA4vCbu7WJmzXOrDF21SjEBz7x2Bm/xisQ90wf5inbCT/dVIiAf4ApbEnkB6qHqsAAAAASUVORK5CYII=': 9
}

answer_num_list = []

# 根据题目的接口,获取base64图片编码信息 每张的图的base64编码是固定的
def get_base64_data(page):
    url = f'http://match.yuanrenxue.com/api/match/4?page={page}'
    headers = {'User-Agent': 'yuanrenxue.project'}
    response = requests.get(url, headers=headers)

    return response.json()['info'], response.json()['key'], response.json()['value']


# 利用key和value计算出属性为display = none 的md5索引值
# 此函数与题目中的ajax这段代码功能一样   hex_md5(btoa(data.key + data.value).replace(/=/g, ''))
def get_j_key(key, value):
    date_str = (key + value).encode('ascii')
    date_str = base64.b64encode(date_str).decode('utf-8').replace('=', '')
    md5 = hashlib.md5(date_str.encode('utf-8')).hexdigest()
    return md5


# 使用正则提取每页中,每一组数字相对应的所以子图片相关数据
def parse_nums(info):
    rule = re.compile(r'<td>(.*?)</td>')
    nums_list = rule.findall(info)
    return nums_list


# 根据j_key和每个图片对应的密文值,确定出要被用的所以数字子图片,和相对位置的偏移量
def parse_num_info(nums,j_key): # 模拟js的过程，对传进来的图片进行筛选
    img_number_list = re.compile(r'img_number (.*?)"').findall(nums) # 获取接口传来的所有class属性的密文值
    base64_img_list =  re.compile(r'base64,(.*?)"').findall(nums) # 获取base64的值
    number_style_list = re.compile(r'style="(.*?)"').findall(nums) # 获取css的偏移值
    display_base64_data = [base64_img_list[index] for index,img_md5 in enumerate(img_number_list) if nums != img_md5] # 获取真正显示的bas64值
    num_list = [image_dict[i] for i in display_base64_data] # 通过代码开头定义有映射关系的字典,解出数字
    offset_list = [number_style_list[index].replace('left:', '').replace('px', '') for index, img_number in enumerate(img_number_list) if img_number != j_key] # 寻找真正显示的子图的偏移量

    true_order_num = get_correct_order(num_list, offset_list)
    answer_num_list.append(true_order_num)

# 此函数用于算出正确排列后的数字
def get_correct_order(num_list, offset_list):
    offset_list = [int(float(i) / 11) for i in offset_list]
    true_order_list = [None] * len(offset_list)
    for index, offset in enumerate(offset_list):
        true_order_list[int(index + offset)] = str(num_list[index])
        # print(true_order_list)
    true_order_num = ''.join(true_order_list)
    return int(true_order_num)


if __name__ == '__main__':
    for i in range(1, 6):
        info, key, value = get_base64_data(i)
        j_key = get_j_key(key, value)
        image_base64_list = parse_nums(info)
        for j in image_base64_list:
            parse_num_info(j,j_key)
    print(f'五页数字总和:{sum(answer_num_list)}')

```

## 总结
本题感觉代码有一定难度，通过迭代器enumerate来分别遍历索引和对应的值，而且在数据中，实际上共用同一个index，关联了好几个列表。最后的还原算法，就是本身的索引加上本身的值得到的就是真实的列表索引，值为原始列表对应的索引。每个功能函数的结果以返回值的形式，通过形参与实参的传递，案例很好，值得研究。
