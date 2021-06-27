# CSDN图片一键下载去水印




### 前言
> 在[吾爱论坛](https://www.52pojie.cn/thread-1367497-1-1.html)上逛逛，看到有个悬赏贴下载CSDN去水印。好像不难，就去试试看。于是今天又可以水一篇博客了，<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210207121025.png"/>

### 代码实现
代码实现很简单。贴主说了，水印和原图的区别就是url后面多了一串东西，那我们只要获取所有的图片地址，然后利用正则过滤一下多的东西，套个循环，下载就行了。
```python
import requests
from lxml import etree
import re
import os
 
 
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36",
    "Referer": "https://blog.csdn.net"}
 
path = os.path.dirname(__file__)
PATH = os.path.join(path,'img').replace('\\','/')
if not os.path.exists(PATH):
    PATH = os.mkdir(PATH)
print("下载路径："+str(PATH))
 
def get_img_data(url):
    img_url = []
    res = requests.get(url, headers=headers).text
    data =  etree.HTML(res).xpath('//*[@id="content_views"]/p/img/@src')
    print(f"一共有{len(data)}张图片")
    for i in data:
        info= re.compile('(.*?)\?').findall(i)[0]
        img_url.append(info)
    return img_url
 
def download_img(data):
    count = 0
    for i in data:
        res = requests.get(i,headers=headers).content
        count += 1
        path = os.path.join(PATH, f'{count}.jpg').replace('\\', '/')
        with open(path, 'ab')as f:
            f.write(res)
            print(f"第{count}张图片下载成功！")
 
if __name__ == '__main__':
    url = "https://blog.csdn.net/yht6005/article/details/107568596?ops_request_misc=%25257B%252522request%25255Fid%252522%25253A%252522161265977516780261989880%252522%25252C%252522scm%252522%25253A%25252220140713.130102334..%252522%25257D&request_id=161265977516780261989880&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-107568596.first_rank_v2_pc_rank_v29&utm_term=VRay4.2%25E5%25AE%2589%25E8%25A3%2585%25E6%2595%2599%25E7%25A8%258B"
    download_img(get_img_data(url))
```
以上代码应该是通用所有CSDN文章通用的，因为网站的结构是相同的，但我没做测试，也说不好。

### 水印参数分析
我们看一下区别
![有水印](https://img-blog.csdnimg.cn/20200724201139172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lodDYwMDU=,size_16,color_FFFFFF,t_70)

![无水印](https://img-blog.csdnimg.cn/20200724201139172.png)
我们对比一下url，这是原来的
`https://img-blog.csdnimg.cn/20200724201139172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lodDYwMDU=,size_16,color_FFFFFF,t_70`  
`https://img-blog.csdnimg.cn/20200724201139172.png`
加了水印之后的网址多了几个参数。分别是`process`,`type`,`shadow`,`text`,`size`,`color`
有些参数看着好像被加密了。看到text的值有一个`=`号，嗯？？？怎么这么像base64加密，拿到解密网站，果然，解码之后是字符串`https://blog.csdn.net/yht6005`，这个就是水印内容了呀。那我们是不是可以随便改了，试试改成我真帅看看。。。
![有水印](https://img-blog.csdnimg.cn/20200724201139172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,text_5oiR55yf5biF,size_16,color_FFFFFF,t_70)
嘿嘿，有意思
经过一系列测试，得到这些参数的作用如下

|  参数   | 作用                               |
| :-----: | :--------------------------------- |
|  type   | 显示的字体，具体怎么加密的不太清楚 |
|  text   | 水印的内容，必须是base64编码之后的 |
|  color  | 水印的颜色                         |
|    t    | 水印的透明度，越高越不透明         |
|  size   | 水印的比例大小，调高即放大水印     |
| shadow  | 阴影填充，数值越高，填充越明显     |
| process | 告诉你这是一个添加水印的图片       |



