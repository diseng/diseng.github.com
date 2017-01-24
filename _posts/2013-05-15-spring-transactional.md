---
layout: post
title : Spring Transactional 测试
category : program
---
{% include JB/setup %}

使用Spring管理事务的时候,只需要在事务类或者事务方法上加上@Transactional注解即可,用着非常方便,但是从来没去验证过加上这个注解是否真的有事务管理的能力.

下面是我测试Spring事务管理的一个流程.

### 1.建表

```sql
create table student
(
   sid                  int not null auto_increment,
   username             varchar(10),
   primary key (sid)
);
```

我建立了一个学生表,只有sid和username两个字段,为了便于测试,我将username长度设置为10.

### 2.Service方法

```java
@Transactional
public void addStudent(Student stu) {
    stuMap.insertSelective(stu);
}
```

这是一个添加学生的方法,其中stuMap是Student的DAO操作类.

### 3.测试方法

```java
@Test
public void testAddStudent(){
    Student s = new Student();
    s.setUsername("test");
    stu.addStudent(s);
    s.setUsername("testlongusername");
    stu.addStudent(s);
}
```

测试方法中调用了2次addStudent方法,其中第二次的username长度大于10,插入将会失败.

运行测试方法,确实报错了.

    ### Error updating database.  Cause: com.mysql.jdbc.MysqlDataTruncation: Data truncation: Data too long for column 'username' at row 1

查看数据库内容

    +-----+----------+
    | sid | username |
    +-----+----------+
    |   1 | test     |
    +-----+----------+

这是否意味在Spring事务管理没有起作用?仔细一看,是我的测试类出了问题,因为Spring管理的是addStudent这个事务,而测试方法中调用了2次addStudent方法,其事务是独立的,第二个addStudent的调用出错,并不会影响第一个addStudent的执行.

### 4.修改Service和测试方法

```java
@Transactional
public void addStudent(Student stu) {
    stuMap.insertSelective(stu);
    Student s = new Student();
    s.setUsername("testlongusername");
    stuMap.insertSelective(s);
}
```

修改后的addStudent方法,在执行第二次insertSelective方法时,会报错,将会使得整个事务回滚,来修改一下测试方法

```java
@Test
public void testAddStudent(){
    Student s = new Student();
    s.setUsername("test");
    stu.addStudent(s);
}
```

执行测试方法,依然报出步骤3中的错误,查看数据库内容

    +-----+----------+
    | sid | username |
    +-----+----------+
    |   1 | test     |
    +-----+----------+

说明Spring确实起到了事务管理的作用,当执行第二次insertSelective方法时,由于出错,回滚了整个事务,所以对数据库没有增加内容.

### 5.去掉@Transactional

步骤4中确实有了事务管理作用,那去掉@Transactional注解,情况又会怎么样呢?

```java
//@Transactional
public void addStudent(Student stu) {
    stuMap.insertSelective(stu);
    Student s = new Student();
    s.setUsername("testlongusername");
    stuMap.insertSelective(s);
}
```

执行步骤4中的测试方法,依然会报出步骤3中的错误,查看数据库内容

    +-----+----------+
    | sid | username |
    +-----+----------+
    |   1 | test     |
    |   3 | test     |
    +-----+----------+

从数据库的内容来看,去掉@Transactional注解后,确实没有了事务管理作用,所以插入了一条新的记录.

注意新插入那条记录的sid为3,而不是2,这是因为步骤4中第一次执行insertSelective时就将sid=2分配了出去,但是第二次执行insertSelective时,由于出错引起了事务回滚,虽然最终没有内容插入,但是已经使用了sid=2,所以下一次插入时,sid就为3了.


