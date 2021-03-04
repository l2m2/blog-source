---
title: 初识SQLAlchemy
toc: true
date: 2021-03-03 09:16:00
description: ...
tags:
- Python
- ORM
- SQLAlchemy
---

## ORM

ORM ( 对象关系映射 Object Relational Mapper )是一种程序设计技术，用于实现面向对象编程语言中里不同类型系统的数据之间的转换。它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”。

简单的说，ORM是通过使用描述对象和数据库之间的映射的元数据，将程序中的对象自动持久化到关系数据库中。

**优点**

1. 简洁易读
2. 可移植：对不同数据库的操作一致
3. 安全：避免SQL注入

**缺点**

1. 性能可能不及原生SQL

## SQLAlchemy

SQLAlchemy提供了一种将用户定义的Python类与数据库表相关联的方法，并将这些类的实例与它们对应的表中的行相关联。

### 安装

```shell
$ pip install sqlalchemy
```

### 版本

```shell
>>> import sqlalchemy
>>> sqlalchemy.__version__
'1.3.22'
```

## 创建Engine

```python
from sqlalchemy import create_engine

engine = create_engine('postgresql://postgres:postgres@localhost/fastapi-vue-admin', echo=True)
```

create_engine的第一个参数是连接字符串，它的格式是：

`${数据库类型}://${数据库用户名}:${密码}@${服务器地址}:${端口}/数据库名称`

端口如果不指定，使用默认端口。

第二个参数是一个打印标志，如果为True, 则会打印SQL语句。

**创建好engine后，数据库连接池也创建好了，但此时还没有真正连接数据库。** 执行`Engine.connect()`时才会真正连接。

## 定义模型

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Boolean, Column, Integer, String

Base = declarative_base()

class User(Base):
  __tablename__ = "sys_user"

  id = Column(Integer, primary_key=True, index=True)
  username = Column(String, unique=True, index=True, nullable=False)
  fullname = Column(String)
  email = Column(String)
  password = Column(String, nullable=False)
  is_active = Column(Boolean(), default=True)
  is_superuser = Column(Boolean(), default=False)
  
  def __repr__(self):
    return f"<User(username = {self.username}, fullname = {self.fullname})>"
```

1. `__tablename__`对应数据库的表名
2. `Column`对应数据库的列
3. `__repr__`是可选的。通常为了在开发和调试时为了更好的显示对象而定义。

## 生成数据库表

```python
Base.metadata.create_all(engine)
```

通过日志可以看到对应的SQL。

```sql
CREATE TABLE sys_user (
        id SERIAL NOT NULL, 
        username VARCHAR NOT NULL, 
        fullname VARCHAR, 
        email VARCHAR, 
        password VARCHAR NOT NULL, 
        is_active BOOLEAN, 
        is_superuser BOOLEAN, 
        PRIMARY KEY (id)
)
```

在实际项目中，我们大概率会使用其他工具来进行数据库表的创建，比如[alembic](https://alembic.sqlalchemy.org/en/latest/).

## 创建Session

```python
session = sessionmaker(bind=engine)
session = Session()
```

Session与上面创建的Engine相关联，但还没有打开任何连接。首次使用时，它会从Engine维护的连接池中获取连接，并一直保留到我们提交所有更改或关闭Session为止。

## CRUD

### 增

```python
user1 = User(username="leon", fullname="李黎明", email="l2m2lq@gmail.com", password="123456")
user2 = User(username="ronaldo", fullname="罗纳尔多", email="ronaldo@gmail.com", password="123456")
session.add(user1)
session.add(user2)
session.commit()
```

可以一次性插入多条，像下面这样

```python
session.add_all([
    User(username="cat", fullname="王二猫", email="cat@gmail.com", password="123456"),
    User(username="dog", fullname="张晓狗", email="dog@gmail.com", password="123456"),
    User(username="pig", fullname="刘大猪", email="pig@gmail.com", password="123456")
])
session.commit()
```

### 查

```python
items = session.query(User).order_by(User.id)
  for item in items:
    print(repr(item))
# Output:
# <User(username = leon, fullname = 李黎明)>
# <User(username = ronaldo, fullname = 罗纳尔多)>
# <User(username = cat, fullname = 王二猫)>
# <User(username = dog, fullname = 张晓狗)>
# <User(username = pig, fullname = 刘大猪)>
```

只查固定列

```python
for username in session.query(User.username):
    print(username)
# Output:
# ('leon',)
# ('ronaldo',)
# ('cat',)
# ('dog',)
# ('pig',)
```

给username一个别名

```python
for row in session.query(User.username.label("name_label")).all():
    print(row.name_label)
```

> *注意*：虽然上面两个例子打印的结果都一样，不同的是.query()返回的是Query对象，而.query().all()返回的是一个列表。

按username降序排序

```python
for row in session.query(User).order_by(User.username.desc()).all():
    print(row)
```

查看姓李的人

```python
for row in session.query(User).filter(User.fullname.like("李%")).all():
    print(row)
```

查找username为leon的人

```python
for row in session.query(User).filter(User.username == 'leon').all():
    print(row)
```

查看username为三个字符的按照username升序后的第一个：

```python
from sqlalchemy.sql.expression import func
row = session.query(User).filter(func.length(User.username) == 3).order_by(User.username).first()
```

>*注意*: all() 返回匹配的所有数据的列表。first()返回第一条，相当于limit 1。one()假定匹配的只有一条，如果有多条或没有，会抛出异常。one_or_none() 假定只有一条，或者没有。其他情况抛出异常。scalar() 和one()类似，但只返回结果中的第一列，且在没有数据时返回None。

### 改

```python
row = session.query(User).filter(User.username == "pig").one()
row.username = 'pig1'
session.commit()
```

### 删

```python
row = session.query(User).filter(User.username == "leon").one()
session.delete(row)
session.commit()
```

## Reference

- https://docs.sqlalchemy.org/en/13/orm/tutorial.html

- https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session

