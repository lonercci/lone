# 文字转语音demo


###  前言
逛吾爱论坛发现有个文字转语音的软件非常火。题者说爬的是收费的接口，下载下来试了一下，震惊了，，语音效果非常棒，很接近人声。但是软件是用易语言写的，而且作者也没有开源，我也不知道他是咋实现的。导致对接口就更好奇了。

### 获得接口

这里使用HTTPDebugger来抓取此软件对外发送的数据请求。我们发现接口的url在电脑端是打不开的，F12模拟手机也一样。于是我们观察请求头，发现有很多的参数，比如referer，host，cookie，UA等等，经过测试，这些参数都必须使用才能正确访问此接口。访问会发现提示参数不正确，我们继续观察这个软件的post请求发送的内容`req=····`后面的内容是使用url编码的。
我们把他复制下来，随便找个解码网站试一下，解码就是一段json数据。具体参数内容还有待研究，先放一放。观察json有个字段是text，对应的就是我们输入的文字，那就好办了，传入text，编码，发送请求，解析响应。在响应中有url字段，就是我们需要的下载链接。后台响应处理的速度还是很快的。拿到链接，下载就大功告成了。
### 代码实现
```python
import requests
import urllib
import re
import time

def get_data(text):
    json = {
              "qd": "110774",
              "ver": "010178",
              "zbid": "4b1ab4d1db435532",
              "text": text,
              "speed": 0,
              "pitch": 0,
              "volume": 95,
              "isurl": 2,
              "viptype": "0",
              "bgdelaytime": 0,
              "textdelaytime": 0,
              "bgvolume": 20,
              "emotion": "",
              "emotiondegree": 50,
              "opertype": "1001"
            }
    str_json=str(json).replace('\'','\"')
    info = urllib.parse.quote(str_json)
    data = "req=" + info
    return data

def get_json(data):
    url = "" ##接口就不放了。。。
    headers = {
        "Referer": "https://servicewechat.com/wx976547f8ab927081/6/page-frame.html",
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 "
                      "Safari/537.36 MicroMessenger/7.0.9.501 NetType/WIFI MiniProgramEnv/Windows WindowsWechat",
        "Host": "pysq.shipook.com",
        "Cookie": "SESSION=YTcyNmU3NmYtZmNkYy00N2IwLTg3YzEtMGRlZWQxMTZlMjUy",
        "Content-Type": "application/x-www-form-urlencoded",
        "Accept": "text/html, application/xhtml+xml, */*",
        "Accept-Encoding": "gzip, deflate",
        "Accept-Language": "zh-cn",
        "Cache-Control": "no-cache",
        "Connection": "Keep-Alive",
        "Content-Length": "401",
    }
    res = requests.post(url, headers=headers, data=data, verify=False).text
    url = re.findall('"audiourl":"(.*?)"',res)[0]
    # print(url)
    return url

def down_url(url):
    headers ={"User-Agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, "
                           "like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"}
    res = requests.get(url,headers=headers).content
    with open(".mp3",'ab') as f:
        f.write(res)
    print("在当前目录下已经下载成功！")

if __name__ == '__main__':
    text = input("请输入你想转成语音的文字，按回车结束:\n")
    print("==================")
    print(text)
    print("==================")
    print("马上就好，请等待下载。。。。")

    time.sleep(3)
    data = get_data(text)
    url = get_json(data)
    down_url(url)

```
### 总结

1.  由于是从别人开发的软件上抓的接口，所以功能比较局限，很多的参数意义不明。
2.  url转码的问题：导入urllib库，使用`urllib.parse.quote()`方法把字符串进行url编码。
3.  下午做了一个签到的云函数，里面有个unicode编码转str的。`res.encode().decode('unicode_escape')`
4.  下载的时候使用`ab`方法写入，如果下载的文件名相同，不会进行覆盖写入，而是接着原来的继续写入。
5.  代码写的很简陋，马马虎虎，不忍直视。


