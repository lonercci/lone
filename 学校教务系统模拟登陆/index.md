# 学校教务系统模拟登陆


## 登录模拟
### 关于
   想尝试一下学校的官网，试试看。
###  登陆页面
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20201209170117.png" style="zoom:67%;" />
   学校的教务系统是青果公司的，很多版块页面都是单独的iframe。但是审查元素单独去访问显示没有权限。
   登陆要求：账号，密码，验证码（以前好像是必须要输入验证码的，现在在一定输入次数之内，可以不用输入。）

### 密码部分
   随便输个错误的账号跟密码
   <img src="https://gitee.com/lonercci/picbed/raw/master/img/20201209171530.png" style="zoom:50%;" />
   发现一堆乱起八早的参数，经过测试，参数作用如下：

| 参数                     | 说明             |
| ------------------------ | ---------------- |
| **__VIEWSTATE**          | html中的         |
| **__EVENTVALIDATION**    | html中的         |
| **dsdsdsdsdxcxdfgfg**    | 加密之后的密码   |
| **fgfggfdgtyuuyyuuckjg** | 加密之后的验证码 |
| **txt_asmcdefsddsd**     | 输入的账号       |
| **txt_pewerwedsdfsdff**  | 输入的密码       |
| **txt_psasas**           | 验证码           |

   密码加密肯定是js完成的，于是找js部分
   鼠标点到输入框那，发现失去焦距和按键弹起会触发shtitblur()和chkpwd()函数，全局搜索，第一个没啥用，第二个出来了，正好是加密部分。
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20201209172835.png" style="zoom:50%;" />

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20201209173354.png" style="zoom:50%;" />

```js
function chkpwd(obj) {
                if (obj.value != '') {
                    var s = md5(document.all.txt_asmcdefsddsd.value + md5(obj.value).substring(0, 30).toUpperCase() + '12810').substring(0, 30).toUpperCase();
                    document.all.dsdsdsdsdxcxdfgfg.value = s;
                } else {
                    document.all.dsdsdsdsdxcxdfgfg.value = obj.value;
                }
            }
```
   这段js的意思是将密码用md5加密，取前30位转大写，然后与密码和学校代码和账号拼接，再进行md5加密取前30位转大写。验证码与密码差不多。
   Python实现：

   ```python
#js中的md5加密实现
def md5(str):
    m = hashlib.md5()
    m.update(str.encode("utf-8"))
    return m.hexdigest()

#登录密码加密(dsdsdsdsdxcxdfgfg)
def login_psd(userid,psd):
    global dsdsdsdsdsdxcxdfgfg
    dsdsdsdsdxcxdfgfg = md5(userid+md5(psd)[:30].upper()+'12810')[:30].upper()
    print(dsdsdsdsdsdxcxdfgfg)

#登录验证码加密(fgfggfdgtyuuyyuuckjg)
def login_yzm(verty_text):
    fgfggfdgtyuuyyuuckjg = md5(md5(verty_text.upper())[:30].upper()+"12810")[:30].upper()
    print(fgfggfdgtyuuyyuuckjg)
   ```

### cookie部分
   做完上述，我以为我又行了，，，结果打脸，无法访问。。。
   后来发现`cookie`部分有个`ASP.NET_SessionId`字段，这是个啥，每次请求还不一样，好像是随机的。查阅资料发现，`ASP.NET`用来判断`web`会话状态的一个参数，只要访问过页面之后就会保存在`cookies`中。
   每次提交账号密码，都会生成一个新的`cookie`，保持同一个会话，否则无法访问。使用`session_res = requests.session()`保持会话，并拿到`ASP.NET_SessionId`的值。

   ```python
   session_res = requests.session()
    url = "http://218.22.58.76:2346/_data/login_home.aspx"
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36"}
    html = session_res.get(url,headers=headers).text
    session_id = session_res.cookies["ASP.NET_SessionId"]
   ```
  拼接`header`
  ```python
  VIEWSTATE = re.findall(r'name="__VIEWSTATE" id="__VIEWSTATE" value="(.*?)"',html)[0]
    EVENTVALIDATION = re.findall(r'name="__EVENTVALIDATION" id="__EVENTVALIDATION" value="(.*?)"',html)[0]
  ```
  最后，拼接`headers`和提交的参数`data`，使用post发送请求，成功登录。
  ~~后续可能会尝试进行课表的爬取，，，，，~~

## 课表爬取
### 目标网页
   url:http://218.22.58.76:2346/ZNPK/KBFB_ClassSel.aspx
   ![](https://gitee.com/lonercci/picbed/raw/master/img/20201221210342.png)

### 抓包分析
   发现必须要输入验证码，而验证码里`headers`里面的`cookie`有两个字段,其中的`sessionid`获取方式上文提到了。首先访问一遍课表入口地址，通过`session`方法获取`cookie`字段，拿到值。
   <img src="https://gitee.com/lonercci/picbed/raw/master/img/20201221210649.png" style="zoom:50%;" />
   输入验证码，继续看发送了什么。`post`请求，发送了专业班级，学期，格式等等，但是这些不是重点。它还把验证码发过去了。返回的请求却不是课表，而是一段`html`。这就很奇怪，后面的那个`get`请求才是对的课表图片。
   验证码正确：
 <img src="https://gitee.com/lonercci/picbed/raw/master/img/20201221212608.png" style="zoom:50%;" />
   验证码错误：
   <img src="https://gitee.com/lonercci/picbed/raw/master/img/20201221212526.png" style="zoom:50%;" />
   这段`html`是组成课表的头部，显示学校专业班级，好像跟课表没什么关系。但是，当验证码输错时，这个请求会弹出`alert`("验证码错误")，没有后续的课表图片请求了。
   由此，`http://218.22.58.76:2346/ZNPK/KBFB_ClassSel_rpt.aspx` 的请求是为了判断验证码是否正确。正确则可以进行后续的请求，错误则警告，后续停止。
   接着`get`一下目标图片的地址，显示全为乱码，`content`一下转二进制流即可。
   <img src="https://gitee.com/lonercci/picbed/raw/master/img/20201221212730.png" style="zoom:50%;" />
   重点在于使用相同`cookie`保持同一个会话(当时这里参数弄错了，困扰了我好久。。。)，还有就是验证码的作用，必须先提交`post`之后，`get`图片才有数据。

### python实现
```python
import requests


url_rk = "http://218.22.58.76:2346/ZNPK/KBFB_ClassSel.aspx"  #课表入口
url_yzm = "http://218.22.58.76:2346/sys/ValidateCode.aspx"   #验证码
url_zz = "http://218.22.58.76:2346/ZNPK/KBFB_ClassSel_rpt.aspx" #验证码确认后的中转链接，返回html课表头部信息
url_kcb = "http://218.22.58.76:2346/ZNPK/drawkbimg.aspx?type=1&w=1110&h=4480&xn=2020&xq=0&bjdm=2017090101" #任意一个课表链接

def get_cookie():
    global cookies
    session_res = requests.session()
    session_res.get(url_rk)
    SessionId = session_res.cookies["ASP.NET_SessionId"]
    # print(SessionId)
    cookies = {
        'myCookie': '',
        'ASP.NET_SessionId': SessionId,
    }

def get_yzm():
    header_yzm = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0',
        'Accept': 'image/webp,*/*',
        'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
        'Connection': 'keep-alive',
        'Referer': 'http://218.22.58.76:2346/ZNPK/KBFB_ClassSel.aspx',
        'Pragma': 'no-cache',
        'Cache-Control': 'no-cache',
       }
    response = requests.get(url_yzm, headers=header_yzm,cookies=cookies)
    with open('.png','wb') as f:
        f.write(response.content)
        print("yzm下载完成，请查看")

def download():
    yzm = input("请输入验证码")
    data = {
      'Sel_XNXQ': '20200',
      'txtxzbj': '',
      'Sel_XZBJ': '2017090101',
      'type': '1',
      'txt_yzm': yzm
    }
    header_kb = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'http://218.22.58.76:2346',
        'Connection': 'keep-alive',
        'Referer': 'http://218.22.58.76:2346/ZNPK/KBFB_ClassSel.aspx',
        'Upgrade-Insecure-Requests': '1',
        'Pragma': 'no-cache',
        'Cache-Control': 'no-cache',
        }
    html = requests.post(url_zz, headers=header_kb, data=data,cookies=cookies).text
    # print(html)
    kcb = requests.get(url_kcb,headers=header_kb,data=data,cookies=cookies).content
    # print(kcb)
    with open('kcb.png','wb')as f:
        f.write(kcb)
        print("kcb完成")

get_cookie()
get_yzm()
download()


```
