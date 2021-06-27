# 使用Pandas合并表格



###  背景
室友因工作需要，要导出大创网的互联网+项目后台学校里面的详细数据。在官网只能导出当前页面的所有项目，一页只显示20个项目。学校总共有1842个，93页的数据，如果一页一页点击导出不太现实。所以想到使用Python来实现。（小声bb）那么大的一个网站，竟然没有考虑一键导出全部详细数据的功能，，，绝了。

### 分析流程
1、打开大创网后台，发现页面内容是通过js动态加载的。网址不会随着页码的变化而变化。
果断抓包，通过url寻找接口
第一页：`https://cy.ncss.cn/contestmanage/contestCount?provinceCode=&schoolCode=&schoolType=&provinceAward=&keyWord=&trackCode=&stage=-9&award=&industryCode=&groupCode=&progress=&isInvest=&pageIndex=0&pageSize=20&diplomaLevelCode=&_=1603205904460`

第二页：`https://cy.ncss.cn/contestmanage/contestList?provinceCode=&schoolCode=&schoolType=&provinceAward=&keyWord=&trackCode=&stage=-9&award=&industryCode=&groupCode=&progress=&isInvest=&pageIndex=1&pageSize=20&diplomaLevelCode=&_=1603205904463`

![](https://s1.ax1x.com/2020/10/20/B9ElnK.png)

![](https://s1.ax1x.com/2020/10/20/B9Eu11.png)

![](https://s1.ax1x.com/2020/10/20/B9EnpR.png)

2、经过对url删减测试，发现pageSize用于控制页面显示项目数量。pageIndex用于显示页数。后面其余参数都可省略。

最终拿到url为`https://cy.ncss.cn/contestmanage/contestList?provinceCode=&schoolCode=&schoolType=&provinceAward=&keyWord=&trackCode=&stage=-9&award=&industryCode=&groupCode=&progress=&isInvest=&pageIndex={}&pageSize=20`

3、在审查元素中，发现下载页面的url很有意思，下载直接就是.xlsx的文件。`https://cy.ncss.cn/contestmanage/exportcontestdetails?contestIds=`后面跟了很多串特定规格字符串，`contestIds=`这个很像是项目的`id`。果不其然，通过审查元素，`xpath`定位，每一个`tr`标签里的值就是`id`。而这个网址传入多少`id`，那么下载的文件里面就有`id`对应的项目。经过测试，60个`id`为最大上限，不然就直接报错。
![](https://s1.ax1x.com/2020/10/20/B9EK6x.png)

![](https://s1.ax1x.com/2020/10/20/B9EMX6.png)
4、拿到下载地址，写个循环，结束下载，可是下载完成后由93个表格数据。
5、使用`os`对文件夹下的93个表格进行循环，拿到每个数据的路径，再通过`pandas`库合并其余表格到一个新表格,结束。

### 代码
1、下载全部数据表格

```python
import requests
from lxml import etree
import os

headers = {
    "cookie":"",
    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36"}

path = r"C:\Users\lenovo\Desktop"
PATH = os.path.join(path,'css').replace('\\','/')
print(PATH)
if not os.path.exists(PATH):
    os.mkdir(PATH)
print("下载路径："+PATH)

for i in range(0,93):
    idss = ""
    url  = "https://cy.ncss.cn/contestmanage/contestList?provinceCode=&schoolCode=&schoolType=&provinceAward=&keyWord=&trackCode=&stage=-9&award=&industryCode=&groupCode=&progress=&isInvest=&pageIndex={}&pageSize=20".format(i)
    r = requests.get(url,headers=headers).text
    ids = etree.HTML(r).xpath("/html/body/table/tbody//tr/@data-value")
    for a in ids:
        idss = idss + a +","
    try:
        data_url = "https://cy.ncss.cn/contestmanage/exportcontestdetails?contestIds=" + idss
        # print(data_url)
        r = requests.get(data_url, headers=headers).content
        # time.sleep(0.5)
    except Exception as e:
        print(e)
        pass
    print("开始下载第{}页".format(i+1))
    print("完成")
    try:
        path = os.path.join(PATH, f'{i+1}.xlsx').replace('\\', '/')
        with open(path, 'wb')as f:
            f.write(r)
            f.flush()
    except Exception as e:
        print(e)
        print("下载失败")
```
2、合并表格
```python
import os
import pandas as pd

list = os.listdir(r'C:\Users\lenovo\Desktop\css')
# print(list)
ov_xlsx = pd.DataFrame()
for info in list:
    domain = os.path.abspath(r'C:\Users\lenovo\Desktop\css')
    info = os.path.join(domain,info)
    # print(info)
    df = pd.read_excel(info,dtype=object)
    ov_xlsx = ov_xlsx.append(df, ignore_index=True)
writer = pd.ExcelWriter(r'C:\Users\lenovo\Desktop\处理后数据.xlsx', index=False,dtype=object)
ov_xlsx.to_excel(writer, index=False)
writer.save()
```

### 总结
总体思路不复杂，当时测试接口数据花了一部分时间，主要是查询pandas库的用法。

### 资料参考

参考知乎大佬： https://zhuanlan.zhihu.com/p/97869613



（此文章由老博客迁移，原链接：http://www.desky2020.xyz/index.php/2020/10/20/22-50/）
