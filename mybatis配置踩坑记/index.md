# MyBatis配置踩坑记


记录一下初学MyBatis的踩坑过程。。。。

>   鬼知道我经历了多少次绝望。。。调试了快三天，终于搞定了第一个MyBatis程序，都是配置惹的祸。。

注意点：

+   我使用的是jdk1.8+IDEA2020。

+   Maven 版本3.6.1。

+   依赖版本：

```xml
<!--导入依赖-->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

```

>   此处经历过绝望。。。。mysql驱动一定要选5.1.49版本的。46,47版的都不行，运行后打印数据库会出现乱码。49版本的useSSl一定要设置为false，否则会一直跳出警告，如果不填则默认为true。我人直接裂开。。。

+   关于mybatis-congfig.xml的配置：

    在xml文档中`;`需要转义，只能填写`&amp;`,等价于`;`。
    
    一定要写`<mappers>`，否则会一直报错，提示没有把接口和类绑定。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/lone/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

+   关于Maven的资源过滤问题：

    在测试类中，有些main或java下的配置文件写不进去。需要放行。

    在.pom中加入以下代码：
```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
</build>
```

哭了，坑都快被我踩塌了。😭
