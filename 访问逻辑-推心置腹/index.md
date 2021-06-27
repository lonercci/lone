# 访问逻辑-推心置腹


## 题目
![](https://gitee.com/lonercci/picbed/raw/master/img/20210128232636.png)

> 抓取下列5页商标的数据，并将出现频率最高的申请号填入答案中

## 分析网页
查看`XHR`里面的内容
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128233405.png" style="zoom: 80%;" />
可以很清晰的看到数据来源于第三个请求。  
查看`Preview`发现里面没有内容，尝试访问一下链接，什么都没有。根据经验，肯定没有那么容易拿到数据。这时我们注意到，请求头里面设置了一个`set-cookie`，
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128233651.png" style="zoom:80%;" />
我们使用接口测试工具带着cookie，发送一次get请求，发现返回了一段`<script>`标签，而不是正常的json数据，格式化之后发现好像跟cookie没有什么关系，我们先放着不用管。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128234216.png" style="zoom:80%;" />
继续看前面的几个请求，发现第二个logo也带有cookie属性。
我们访问其他几页试试看。
![](https://gitee.com/lonercci/picbed/raw/master/img/20210128234736.png)
发现了一个有意思的现象，logo跟3这两个请求似乎是绑定到一起的，每次访问下一页，都是先请求logo，在请求3，而且cookie值相同。  
那么我们可以大胆的猜测一下，是否每次访问一个页面，都是先访问一次logo，然后设置了一个cookie值，于是后面的3请求就有了cookie。

## 验证猜想
我们用python代码测试了一下，发现
- 单独访问`http://match.yuanrenxue.com/api/match/3`是得不到数据的，而是一段上面说的代码，并没有什么用处。
- 设置了一个session来保持会话，首先访问logo，再次访问3，可以得到数据。<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210128235931.png" style="zoom:50%;" />
- 通过.cookies方法打印出来看看，果然设置了在第一个请求就设置了cookie。<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210129000138.png" style="zoom:50%;" />

那么代码就好写了。
```python
import requests
# 实例化session
session = requests.session()

# 设置请求头,必须这么写,不然会请求失败,
headers = { 'Connection': 'keep-alive',
            'User-Agent': 'yuanrenxue.project',
            'Accept': '*/*',
            'Origin': 'http://match.yuanrenxue.com',
            'Referer': 'http://match.yuanrenxue.com/match/3',
            'Accept-Encoding': 'gzip, deflate',
            'Accept-Language': 'zh-CN,zh;q=0.9,en-GB;q=0.8,en;q=0.7',
           }
# 名为【logo】请求的url
url = 'http://match.yuanrenxue.com/logo'
# 设置请求头数据
session.headers = headers
# 使用session发起请求
response = session.post(url=url)

url_api = f'http://match.yuanrenxue.com/api/match/3'
res = session.get(url=url_api).json()
# 成功获取到数据,猜想成功
print(res)
#{'status': '1', 'state': 'success',
# 'data': [{'value': 2838}, {'value': 7609}, {'value': 8717},
# {'value': 6923}, {'value': 5325}, {'value': 4118}, {'value': 8884},
# {'value': 8717}, {'value': 2680}, {'value': 3721}]}
```

## python爬取
使用pandas里面的函数value_counts()求取众数 。
```python
import requests
import pandas as pd

def get_data(page_num):
    session = requests.session()

    headers = { 'Connection': 'keep-alive',
                'User-Agent': 'yuanrenxue.project',
                'Accept': '*/*',
                'Origin': 'http://match.yuanrenxue.com',
                'Referer': 'http://match.yuanrenxue.com/match/3',
                'Accept-Encoding': 'gzip, deflate',
                'Accept-Language': 'zh-CN,zh;q=0.9,en-GB;q=0.8,en;q=0.7',
               }
    url = 'http://match.yuanrenxue.com/logo'
    session.headers = headers
    response = session.post(url=url)

    url_api = f'http://match.yuanrenxue.com/api/match/3?page={page_num}'
    res = session.get(url=url_api).json()
    data = [i['value']for i in res['data']]
    return data


if __name__ == '__main__':
    data = []
    for i in range(1,6):
        data_list = get_data(i)
        data.extend(data_list)
    count = pd.value_counts(data)
    print(count)
```
## 总结
本题不难，多用python测试几次就知道了。题目名字叫访问逻辑，顾名思义，应该就是请求访问的先后顺序不能错，否则就算cookie正确也拿不到正确的值。
