# 记录一次本地数据库的制作


### 前言
> 一切因为昨天晚上无意间打开了易班APP。

里面有个轻应用可以直接提交补助申请，点进去看了一眼，发现页面有交互功能（因为以前在易班工作，知道这玩意光凭易班自带的轻应用模块肯定做不了），很好奇是哪位大神做的。于是很快啊，我啪的一下就点进去了，发现登陆之后随便点一个表格，会自动补全很多个人信息。我靠！！！这么强的吗，不会是直接对接学校数据库的吧。赶紧看看。

### 拿到接口地址

刚刚登陆的时候，发现只需要填写姓名跟学号就可以了。但是随便选一个表格却可以补全很多信息，包括，班级，学院，身份证号，出生日期等等。按照这个思路，它应该是提交了你填写的登录表单信息，在服务器进行比对，正确会跳转页面，给你拿到其余信息。

妙就妙在易班的页面有分享功能，这样我们可以直接拿到这个东西的网址。 好家伙，果然是学校的服务器。在浏览器中打开，无法访问，F12手机端模拟一下可以打开了。

很轻易的，可以拿到接口地址，模拟请求一下，发现返回了一段json数据，仔细观察，全部都是我们想要的信息。

### 遍历数据库
我们发现，这样拿到的json数据是自己的。因为我们登陆的是自己的账号。
那么我们只要有其他人的姓名跟学号，那么不就可以把数据从服务器里面遍历出来了吗。
巧的是，我确实有一部分学号+姓名的Excel数据。于是活来了，考虑使用pandas库，解析Excel，拼接参数。发送请求，下载json。
```python
import requests
import re
import pandas as pd



def get_ns():
    df = pd.read_excel("......xlsx")
    # print(df)
    info = df.values
    print(f"++++++++++++本次下载一共用{len(info)}条数据+++++++++++")
    # print(info)
    return info

def get_json(name, studentNo,count):

    url = "https://eclass.wendaedu.com.cn:6006/XXXXX"
    headers = {
        "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"}
    data = {"studentName": name,
            "studentNo": studentNo}
    res = requests.post(url, headers=headers, data=data).text
    if len(res)<50:
        print(f'学号 {studentNo} 的信息有误，无法查询！！！')
    else:
        title_no = re.findall('"loginname":"(.*?)",',res)[0]
        title_class = re.findall('"classes":"(.*?)",',res)[0]
        # with open(f'school/{title_class}--{title_no}.json','w',encoding="utf-8") as f:
        #     f.write(res)

        print(f"======>{studentNo}信息保存成功！")



if __name__ == '__main__':

    count = 0
    for i in get_ns():
        studentNo = str(i[2])
        name = str(i[0])
        print(f"{name}-{studentNo}开始下载======>")
        get_json(name,studentNo,count)

```

这样，其实我们的信息都拿到了，最后解析每一个json，就可以拿到数据。

### 尝试构建一个本地数据库

当然，这并没有完，最近学了数据库，肯定要尝试一下能不能自己弄个本地的小数据库。思路很简单，遍历所有json文件，并解析字段作为表头，然后使用SQL语句插入数据就可以了。附上部分代码：

```python
classes = temp["classes"]  # 班级
stuno = temp["loginUser"]["loginname"]  # 学号
name = temp["loginUser"]["name"]    # 姓名
gender = temp["loginUser"]["gender"]   # 性别
politicalstatus = temp["loginUser"]["politicalStatus"]    # 政治面貌
nation = temp["loginUser"]["nation"]    # 民族
education = temp["loginUser"]["education"]    # 学历
phone = temp["loginUser"]["phone"]  # 联系方式
biogenicland = temp["loginUser"]["biogenicland"] # 生源地
idno = temp["loginUser"]["idno"] # 身份证号码
instructor = temp["loginUser"]["instructor"]  # 辅导员姓名
major = temp["major"]  # 专业
grade = temp["grade"]  # 年级
department = temp["department"]  # 学院
no = temp["loginUser"]["no"]  # 考生号
admissionDate = temp["loginUser"]["admissionDate"]  # 入学日期
email = temp["loginUser"]["email"] # 电子邮件
address = temp["loginUser"]["address"] # 家庭住址
homephone = temp["loginUser"]["homephone"]  # 联系人电话
bankcardnumber = temp["loginUser"]["bankcardnumber"]  # 银行卡号
id = temp["loginUser"]["id"]  # 用户唯一ID
collegeid = temp["loginUser"]["collegeId"]  # 学校ID
periodId = temp["loginUser"]["periodId"]  # 时期ID
majorId = temp["loginUser"]["majorId"]  # 专业ID
classId = temp["loginUser"]["classId"]  # 班级ID
```

竟然有这么多字段，手都要敲断了，有些字段的值为空。

```python
db = pymysql.connect("localhost","root","123456","wenda",charset="utf8")
    cursor = db.cursor()
    # cursor.execute("use wenda") # 使用 wenda 这个数据库
    try:
        sql = f"insert into wenda.student (id,classes,stuno,name,gender,politicalstatus,nation,education,phone,idno,biogenicland,instructor,major,grade,department,no,admissionDate,email,address,homephone,bankcardnumber,collegeid,periodId,majorId,classId)values\
                ('{id}','{classes}','{stuno}','{name}','{gender}','{politicalstatus}','{nation}','{education}','{phone}','{idno}','{biogenicland}','{instructor}','{major}','{grade}','{department}','{no}','{admissionDate}','{email}','{address}','{homephone}','{bankcardnumber}','{collegeid}','{periodId}','{majorId}','{classId}')"
        cursor.execute(sql)
        db.commit()
    except Exception as e:
        print('Insert Failed')
    db.commit()
    cursor.close()
    db.close()
```

套个循环，最终得到了10525条数据。

### 成果
<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210405011251.png" style="zoom:80%;" />

到最后一刻，还是很激动人心的。完结撒花。

最后的最后。。。。（几个小时之后）

发现这个域名是给易班用的。我曾经还用过这个后台，上传过数据。。。如今，我再通过接口爬下来，再做成数据库。。。。？？？这是什么鬼操作？？

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210405012326.png" style="zoom:33%;" />
