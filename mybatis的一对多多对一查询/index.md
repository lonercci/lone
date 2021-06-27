# MyBatis的一对多，多对一查询


### 数据表
现在有两张表，分别是`Student`和`Teacher`。两张表的`id`均为主键，而`Student`表中的`tid`为外键关联到了`Teacher`表的主键。如图：

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210505121905.png" style="zoom:50%;" />

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210505123655.png" style="zoom: 50%;" />

#### 相应的SQL语句

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(id, NAME) VALUES (1, '秦老师');

CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),KEY `fktid` (`tid`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (1, '小明', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (2, '小红', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (3, '小张', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (4, '小李', 1);
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (5, '小王', 1);
```
### 多对一
通过学生表，查询授课老师，属于多对一的关系。
按照我的理解，SQL有如下两种方式书写
```sql
--第一种(子查询)
select s.id,s.`name`,t.`name` 
from student s,teacher t 
where tid=(select id from teacher )

--第二种
select s.id,s.`name`,t.`name`
from student s,teacher t
where s.tid = t.id;
```
图例：

<img src="https://gitee.com/lonercci/picbed/raw/master/img/20210505123500.png" style="zoom:50%;" />

那么在Mybatis中，写好了实体类与接口，该如何对应的写出相应的Mapper.xml。
为了连Teacher表，我们在建立实体类的时候，建了一个Teacher类，这样通过Teacher类可以方便的拿到表里面的数据。

#### 按照查询嵌套处理

```xml
    <select id="getStudent" resultMap="StudentTeacher">
        select * from mybatis.student
    </select>
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>
    <select id="getTeacher" resultType="Teacher">
        select *from mybatis.teacher where id = #{tid}
    </select>
```
使用select进行学生数据表查询时，由于不是一个单一的对象，所以我们使用结果集映射`resultMap`而不是`resultType`。`resultMap`的属性值我们可以随意定义，接着对结果集的每一个字段与属性进行一一映射。`colum`就是数据库中的列名或名别。对于复杂的属性使用`association`进行配置，非常类似嵌套查询。

#### 根据结果查询

```xml
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname,t.name tname,t.id tid
        from mybatis.student s,mybatis.teacher t
        where s.tid = t.id;
    </select>
    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
            <result property="id" column="tid"/>
        </association>
    </resultMap>
```
Teacher中的实体类
```java
@Data
public class Teacher {
    private int id;
    private String name;
}
```

Student中的实体类
```java
@Data
public class Student {
    private int id;
    private String name;

    //学生需要关联一个老师
    private Teacher teacher;
}
```

### 一对多

#### 根据结果查询
```xml
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid,s.name sname,t.name tname,t.id tid
    from mybatis.student s,mybatis.teacher t
    where s.tid = t.id and t.id = #{tid};
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

#### 按照查询嵌套处理

```xml
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="id"/>
    <collection property="students" column="id" javaType="ArrayList" ofType="Student" select="getStudentTeacherId"/>
</resultMap>
<select id="getStudentTeacherId" resultType="Student">
    select * from mybatis.student where tid = #{tid}
</select>
```
Teacher中的实体类

```java
@Data
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;
}
```

Student中的实体类

```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;
```


### 总结

> 各属性名的作用：
> column : 数据库中的列名，或者是列的别名
> property: 映射到结果的字段或者属性
> javaType: 一个java类的完全限定名
> ofType:集合中的泛型信息

1.  关联 - association  【多对一】
2.  集合 - collection     【一对多】
3.  javaType        &          ofType
    1.  JavaType 用来指定实体类中的属性的类型
    2.  ofType 用来指定映射到List或者集合中的 pojo 类型，泛型中的约束类型

