# 从Hexo到Hugo的博客迁移脚本Demo


### 从Hexo到Hugo

从一个框架到另外一个框架的过渡，比较麻烦的就是文章的迁移，尤其是文章数量较多的时候。如果一个一个手动更改 MD 文档的头部配置信息，无疑是一个巨大的工程量。这个时候我们可以考虑使用脚本来完成重复性操作。

观察Hexo和HuGo的文档头部配置，很相近。

![](https://gitee.com/lonercci/picbed/raw/master/img/20210626210130.png)

一个是类似于yaml的配置文件，一个类似于toml的配置文件。

### 思路

我是这样思考的：

+   把`Hexo`的`_post`路径下的所有文章全部复制到新的文件夹（怂，不敢对源文件直接操作）重命名`Old`，继续新建一个文件夹`New`，存放更改好的文件。

+   遍历此目录下的所有文件，使用os的读写操作，截选配置部分，进行解析，拿到单个参数对应的值。

+   把值按照Hugo的格式进行拼接，拿到Hugo的头部配置

+   把头部文件和剩余的正文部分，拼接，写入新的文件中，完成。

### 困境

思路很简单，就是文件字符的拼接罢了。

但是在实际操作中，发现以前博客的头部格式书写的并不规范，有时有好几个`tags`，这种情况使用正则直接提取``---xxxx---``的部分最方便，但是我写不好表达式，无奈只能按行读取按行取大致的前10行左右，通过判读符号`: `是否存在来确定内容是否是头部配置部分。而时间中的：又会影响我使用列表分片取值，而且文章有大量的转义字符等，这些都需要考虑进去.....我哭了。。。乱七八糟写了一堆 `if..else`，终于能把头部配置转换过去了。
结果，正文部分的截取我不知道该如何操作了。有一种思路是通过文件修改前几行数据进行覆盖操作，但是百度了好多方法没成功，而且网上似乎对文件的更改操作教程较少。。。另一种思路是把文件内容加载到内存中，赋给一个变量，截选进行替换，但是强转之后是莫名使用不了`replace`方法，因为不是Str类型，可实际上已经使用`"".join()`方法转换过了。于是我卡在了这个地方，暂时还没有好的想法。。。

### 部分程序
```python
import os

for root, dirs, files in os.walk(r'C:\Users\lenovo\Desktop\Old'):
    for file in files:
        path = os.path.join(root, file)

        with open(path, "r+", encoding="utf-8") as f:
            data = f.readlines()[0:9]
            print(data)
            data_list = []
            tags = []
            for i in data:
                if ":" in i and "tags":
                    info = i.split(": ")
                    data_list.append(info)
                elif "- " in i:
                    i = i.lstrip()
                    tags.append(i[2:-1])
            # print(data_list)
            tag = data_list[2][1].replace("\n", "").lstrip()
            tags.append(tag)
            for a in tags:
                if a == "":
                    tags.remove(a)
            head = "---\n"
            new_tag = "tags" + ": " + str(tags).replace("\'", "\"")
            new_title = "title" + ": " + data_list[0][1].replace("\n", "")
            new_date = "date" + ": " + data_list[1][1].replace(" ", "T").replace("\n", "") + "+08:00"
            new_categories = "categories" + ": " + "[\"" + data_list[3][1].replace("\n", "").lstrip() + "\"]"
            new_imgurl = "featuredImage" + ": " + "\"" + data_list[4][1].replace("\n", "") + "\""
            end = "---\n\n"
            head_last = head + new_title + "\n" + new_date + "\n" + "draft: false\n" + new_tag + "\n" + new_categories + "\n" + new_imgurl + "\n" + end
            print(path)
            print(head_last)

```

![](https://gitee.com/lonercci/picbed/raw/master/img/20210626213257.png)

这里只实现了从`hexo`到`hugo`更改头部配置，但是怎么和正文部分合并，还未完成。
