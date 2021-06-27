# 某网站爬虫练习


### 关于
  - 最近在吾爱破解论坛闲逛，发现有大佬发布了某网站爬虫题分析。很好奇，于是试了一下，发现我是真滴菜，好多类型的反爬措施都没见过，于是写篇博客来记录一些学习的思路过程。
  - 网站链接：http://match.yuanrenxue.com/list
### 第十二题（入门级js）
  题目链接：http://match.yuanrenxue.com/match/12
  F12抓包找到json接口数据：
  ```http://match.yuanrenxue.com/api/match/12?page=1&m=eXVhbnJlbnh1ZTE%3D```
  ```http://match.yuanrenxue.com/api/match/12?page=2&m=eXVhbnJlbnh1ZTI%3D```
  ```http://match.yuanrenxue.com/api/match/12?page=3&m=eXVhbnJlbnh1ZTM%3D```
  对```url```进行解码，发现```%3D```就是```=```。
  两个参数，```page```：页数，```m```：不知道
  找参数```m```，发现太多了。想到以前爬全民k歌的时候在当前html中页找到的js部分，试试看。果然，，，可行。
  ![](https://gitee.com/lonercci/picbed/raw/master/img/20201205200601.png)
  `btoa()`方法用于创建一个`base-64`编码的字符串。
  `base-64` 解码使用方法是 `atob()` 。
  即`yuanrenxue+str(page)`的`base64`加密
  找个`base64`加密的网站测试,第一个`yuanrenxue1`加密后为   `eXVhbnJlbnh1ZTE=`，比对，成功。
  代码：

  ```python
import requests
import base64

headers = {"User-Agent":"yuanrenxue.project"}
sum = 0
for i in range(1,6):
    m= base64.b64encode(("yuanrenxue"+str(i)).encode()).decode()
    url = "http://match.yuanrenxue.com/api/match/12?page={}&m={}".format(i,m)
    response = requests.get(url,headers=headers).json()
    print(response)
    for each in response['data']:
        sum += each['value']
print(sum)

  ```


### 第十三题（入门级cookie）

   题目链接：http://match.yuanrenxue.com/match/13
   这题目我百思不得其解，为什么请求的数据都是空的，加了```cookie```还是无法显示。
   后来询问大佬，才知道这是一道典型的```cookie```反爬类型题目。如果访问页面第一次出现明显的卡顿感，之后浏览器自动刷新了。很有可能就是这种类型。很多的```cookie```处理思路都是，服务器先返回```js```,浏览器自动执行，之后刷新页面再访问。
   果不其然，观察了一下，刚开始所有数据数值都是500，之后数据自动刷新，数值才发生的改变。F12发现有两个名为```13```的数据包先后发送。另外还有一个是名为```13```的数据包是```api```接口数据。观察数据包发现请求```url```都是```http://match.yuanrenxue.com/match/13```，但是```cookie```处发生了改变，增加了```yaunrenxue_cookie```字段，所以，很明显，打开```url```后，返回```js```代码，运行生成```cookie```，给接口`api`使用。
   使用接口测试一下，拿到`js`部分
   ![](https://gitee.com/lonercci/picbed/raw/master/img/20201206123148.png)
   ```javascript
   document.cookie=('y')+('u')+('a')+('n')+('r')+('e')+('n')+('x')+('u')+('e')+('_')+('c')+('o')+('o')+('k')+('i')+('e')+('=')+('1')+('6')+('0')+('7')+('2')+('2')+('9')+('0')+('7')+('3')+('|')+('W')+('o')+('h')+('j')+('1')+('e')+('m')+('J')+('u')+('K')+('k')+('G')+('u')+('Y')+('H')+('X')+('z')+('s')+('4')+('m')+('t')+('A')+('8')+('h')+('u')+('B')+('E')+('b')+('f')+('A')+('X')+('B')+('F')+('t')+('9')+('G')+('A')+('N')+('c')+('d')+('R')+('H')+('r')+('M')+('M')+('F')+('H')+('q')+('L')+('0')+('8')+('G')+('s')+('o')+('f')+('B')+('R')+('V')+('C')+('N')+('u')+('D')+('l')+('f')+('O')+('t')+('l')+('Q')+('K')+('B')+('l')+('H')+('x')+('t')+('c')+('w')+('N')+('1')+('o')+('J')+('9')+('J')+('W')+('N')+('q')+';path=/';location.href=location.pathname+location.search
   ```
   字段为这些字符串相加，用正则，找到`cookie`了,使用`session`来实现,`cookie.set()`方法用来添加`cookie`。

   代码实现：
   ```python
import re
import requests

session = requests.Session()
headers = {"User-Agent": "yuanrenxue.project"}
url = "http://match.yuanrenxue.com/match/13"
r = session.get(url).text
# print(r)

reg = re.compile("'([a-zA-Z0-9=|_])'")
results = reg.findall(r)
# print(results)
cookie =''.join(results)
key,value = cookie.split('=')
# print(key,value)
a = session.cookies.set(key,value)
sum = 0
for i in range(1,6):
    api_url ="http://match.yuanrenxue.com/api/match/13?page={}".format(i)
    r = session.get(api_url,headers = headers)
    data = r.json()
    # print(data)
    values = data["data"]
    for i in values:
        sum += int(i["value"])
print(sum)

   ```

### 第一题（js混淆源码乱码）




