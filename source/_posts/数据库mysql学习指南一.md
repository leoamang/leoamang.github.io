---
title: 数据库mysql学习指南一
date: 2022-06-16 15:28:19
tags:
---

> flask开发有两种模式，一种是 写 model.py 生成数据库表，另一种是在navicat 里建表，反向工程生成 model.py，这个model.py就是把数据表和Python内存中的对象做一个映射，也就是sqlalchemy框架的作用。这样，你操作python内存中的对象就可以实现对数据库的增删改查了   --h老师

# 简单介绍 #

SQLAlchemy 是 Python 著名的 **ORM 工具包**。通过 ORM，开发者可以用**面向对象的方式来操作数据库**，不再需要编写 SQL 语句。SQLAlchemy 支持多种数据库，除 **sqlite** 外，其它数据库需要安装第三方驱动。

SQLAlchemy是Python编程语言下的一款开源软件。提供了**SQL工具包**及**对象关系映射（ORM，Object Relational Mapping）**工具，ORM也就是采用对象关系映射的模式，使用这种模式可以将数据库中各种数据表之间的关系映射到程序中的对象。这种模式可以屏蔽底层的数据库的细节，不需要我们与复杂的 SQL 语句打交道，直接采用操作对象的形式操作就可以。

SQLAlchemy 取得成功的一个证明就是围绕它已建立了丰富的社区。针对 SQLAlchemy 的扩展和插件包括：**declarative、**Migrate、Elixir、SQLSoup、django-sqlalchemy、DBSprockets、FormAlchemy 和 z3c.sqlalchemy。



# 建立与数据库的连接 #

Engine对象是使用SQLAIchemy的起点，根据Engine的构架示意图，Engine包括数据库连接池（Pool）和方言（Dialect，指不同数据库的sql语句等的语法差距），两者把一起对数据库的操作，以符合[DBAPI](https://gitee.com/freakchicken/db-api)（快速生成数据库的http接口服务，零代码开发，只需编写sql，就可以生成http api服务。是数据库的上层应用，方便数据库数据对外发布http服务）规范的方式与数据库交互。                                                                                                    

![image-20220615191554658](数据库mysql学习指南一/image-20220615191554658.png)







## python导入mysql ##

### sqlalchemy使用table、metadata和mapper做映射 ###

####  1下载库 ####

安装sqlalchemy和pymysql库

```
pip install sqlalchemy 

pip install pymysql
```

#### 2 连接数据库 ####

1. ##### 配置信息 #####

   ```
   dialect+dirver://username:passwod@host:port/database,echo=True
   数据库类型+数据库驱动://数据库用户名:数据库密码@IP地址:端口/数据库，其他参数
   ```

   dialect：数据库，如sqlite、mysql、oracle
   dirver：数据库驱动，用于连接数据库的，如果是mysql则使用pymysql
   username：用户名
   password：密码
   host：IP地址
   port：端口
   database：数据库表格名称

   echo:是否进入调试模式

   ```python
   HOST=''
   PORT=
   USERNAME=''
   PASSWORD=''
   DB=''
   
   //dialect+dirver://username:passwod@host:port/database,echo=True
   DB_URL=f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'
   ```

   上述的配置可以放入config.py文件

2. ##### 创建引擎并连接数据库 #####

   ```
   from sqlalchemy import create_engine
   from config import DB_URL
   
   engine = create_engine(DB_URL)  #创建引擎
   conn=engine.connect()  #连接数据库
   result=conn.excute('SELECT 1')   #执行sql语句
   print(result.fetchone())
   conn.close() #关闭连接
   ```

3. ##### 创建ORM模型并映射到数据库 #####

   ```
   from sqlalchemy.ext.declarative import declarative_base
   from sqlalchemy import create_engine,Column,Interger,String
   from sqlalchemy.orm import sessionmaker
   from config import DB_URL
   
   engine =create_engine(DB_URL)
   Base = declarative_base(engine)  #SQL的ORM基类
   session = sessionmaker(engine)()  #构建session对象  也可以写为Session=sessionmaker(engine)  session=Session()
   
   
   class Student(Base):
     __tablename__='student'
     id=Column(Integer,primary_key=True,autoincrement=True)
     name=Column(String(50))
     age=Column(Integer)
     sex=Column(String(10))
     
     Base.metadata.create_all() #将模型映射到数据库
    
   ```

#### 3 对数据库的操作 ####

1. ##### 新增数据（需要提交到数据库） #####

   ```
   student=Student(name='tony',age='18',sex='男')
   session.add(student) #添加到session
   session.commit() #提交到数据库
   
   #批量添加数据
   session_add_all([
          Student(name='张三',age='18',sex='男'),
          Student(name='李四',age='18',sex='女')
   ])
   session.commit()
   ```

2. ##### 查询数据 query() #####

   ```
   item_list = session.query(Student).all()  #item_list是一个包含多个Student对象的列表
   print(item_list)
   for item in item_list:
      print(item.name,item,age)
      
      
   #指定查询列
   item_list=session.query(Student.name).all()
   print(item_list)
   
   #返回数据的第一行
   item=session.query(Student.name).first()
   print(item)
   
   #使用filter()方法进行筛选
   item_list=session.query(Student.name).filter(Student.age>=18).all()
   print(item_list)
   
   #使用order_by()进行排序
   item_list=session.query(Student.name,Stugent.age).order_by(Student.age.desc()).all()  #desc()表示倒序
   print(item_list)
   
   
   #多个查询条件(and 和 or)默认是and
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(Student.age >= 10, Student.sex == 'female').all()
   print(item_list)
   
   ##or
   from sqlalchemy import or_
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(or_(Student.age >= 20, Student.sex == 'female')).all()
   print(item_list)
   
   
   
   #equal/like/in
   #等于
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(Student.age == 18).all()
   print(item_list)
   #不等于
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(Student.age != 18).all()
   print(item_list)
   #like
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(Student.name.like('%To%')).all()
   print(item_list)
   #in
   item_list=session.query(Student.name,Stugent.age,Student.sex).filter(Student.age.in_[16,20]]).all()
   print(item_list)
   
   
   #count 计算个数
   count=session.query(Student).count()
   print(count) 计算数据的条数
   
   
   #切片
   item_list=session.query(Student.name).all()[:2]
   print(item_list)
   ```

3. ##### 修改数据updat（）需要提交到数据库 #####

   ```
   session.query(Student).filter(Student.name == 'Tony').update({'age':22})
   session.commit()
   ```

4. ##### 删除数据delete() #####

   ```
   session.query(Student).filter(Student.name=='Ben').delete()
   session.commit()
   ```

   [sqlalchemy的基本用法 - 蓝莓薄荷 - 博客园 (cnblogs.com)](https://www.cnblogs.com/blueberry-mint/p/14277882.html)

   [Flask-SQLAlchemy详解 - 简书 (jianshu.com)](https://www.jianshu.com/p/f7ba338016b8)

~~sqlalchemy使用delect~~

~~https://www.csdn.net/tags/Ntzagg2sNTA4MDEtYmxvZwO0O0OO0O0O.html~~

~~几种常用插件的数据库url格式：~~

```
MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>

pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]

MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>

cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]
```



2022.6.23
