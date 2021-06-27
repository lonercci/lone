# 全民K歌下载器




### 前言
希望最终的美好都能为你如期而置。

### 获取接口
1、目标：下载指定用户主页的全部歌曲。
2、对比`url`，删除不必要的参数后发现，用户`uid`为个人凭证。
3、分析网页源代码得到用户所有的json数据接口为`https://node.kg.qq.com/cgi/fcgi-bin/kg_ugc_get_homepage?type=get_uinfo&start={}&num=10&share_uid={}`
第一个参数是页码，每页的`json`数据有10条，第二个参数为用户凭证信息。

### 代码实现
```python
import requests
from lxml import etree
import json
import time
import re
import os


headers = {
        "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36",
}
path = os.path.dirname(__file__)
PATH = os.path.join(path,'music').replace('\\','/')
if not os.path.exists(PATH):
    PATH = os.mkdir(PATH)
print("下载路径："+PATH)


def get_data():
     c=0
     uid = input("请输入k歌主页uid：")
     header_url = "https://kg.qq.com/node/personal?uid={}".format(uid)
     r = requests.get(header_url,headers=headers).content.decode()
     user_name = "".join(etree.HTML(r).xpath("//*[@class = 'my_show__name']/text()"))
     music_num = "".join(etree.HTML(r).xpath("//*[@class = 'my_nav__list']/li[1]/a/text()"))[3:]
     num = int(music_num) // 10
     if num != int(music_num) / 10:
         num = num + 2
     print("昵称：{} ——> 一共有{}首音乐,数据共{}页\n".format(user_name,music_num,num-1))

     for i in range(1,num-1):
         try:
             url = "https://node.kg.qq.com/cgi/fcgi-bin/kg_ugc_get_homepage?type=get_uinfo&start={}&num=10&share_uid={}".format(i,uid)
             # print(url)
             print("\n这是第{}页数据：{}".format(i,url))
             res = requests.get(url,headers=headers).text
             time.sleep(1)
             r = json.loads(res[18:-1])
             music_name = r["data"]["ugclist"]
             for a in music_name:
                 title = a["title"]
                 if "|" or "?" or ":" in title:
                     title = title.replace("|", " ")
                     title = title.replace("?", "")
                     title = title.replace(":", "")
                 shareid = a["shareid"]
                 try:
                     print("  "+title+"--> "+"https://kg.qq.com/node/personal?uid="+shareid)
                     print("    正在下载……")
                     music_url = "http://node.kg.qq.com/play?s={}".format(shareid)
                     data = requests.get(music_url,headers).text
                     data_url = re.findall(r'playurl":"(.*?)"', data)[0]
                     res = requests.get(data_url, headers=headers,timeout=(1,30))
                     music = res.content
                     path = os.path.join(PATH, f'{title}.mp3').replace('\\', '/')
                     try:
                         with open(path, 'ab')as f:
                             f.write(music)
                             f.flush()
                         c = c+1
                         print("      【第{}首-——>{}--->下载成功】".format(c,title))
                     except Exception as e:
                         # news = "!!!sorry，{}下载失败!!!,稍后重新下载".format(title)
                         print(e)
                 except:
                     pass
         except:
             pass
get_data()

```
### 运行截图
![](https://s1.ax1x.com/2020/10/14/0I6GFS.png)

### 问题总结
1、由于把歌名作为MP3的文件名，所以歌名中如果出现`"|","?",":"`等特殊符号，会报错，当时还纠结了好长时间，为什么会报错。。。
2、不足之处有一下几点：
   1）如果用户把相同名称的歌曲重复发布，则会造成下载覆盖，只保留一首歌曲。
   2）经过测试，发现下载成功率还是很高的。但是总有些例外没能下载下来，也没做下载失败的歌曲保存再等全体结束后重新下载。
   3）这个程序只能一键下载某个用户的全部音乐，但是如果此用户后续更新了，不能针对更新的内容单独下载，体验性不高。
3、可能会更新，也可能不会，，，

(本文由老博客迁移至此....)

