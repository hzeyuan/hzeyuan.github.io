---
layout: post
title: "python3一些类方法的总结"
subtitle: 'python中一些类的方法'
author: "Hzy"
header-style: text
tags:
  - Python
---

> 最近看了很多这类型的文章，想要自己总结下，顺便补下数据库的知识..数据库学的太烂了。
>

-----
### 1.创建数据库和表
----
#### 1.第一种方式
```
# 同时，创建数据库和表
engine = create_engine('sqlite:///test.db', echo=True)

metadata = MetaData(engine)
Base = declarative_base()


# 第一种方式：使用类来创建一个表
# 同时，创建数据库和表,echo显示sql语句
engine = create_engine('sqlite:///test.db', echo=True)

metadata = MetaData(engine)
Base = declarative_base()


# 第一种方式：使用类来创建一个表
class User(Base):
    # 表的名称
    __tablename__ = 'users'
    id = Column(Integer(), primary_key=True)
    username = Column(String(20), nullable=True, index=True)
    password = Column(String(20), nullable=True)


# 创建
Base.metadata.create_all(engine)
```
### 第二方式，使用Table
```
engine = create_engine('sqlite:///test.db', echo=True)

metadata = MetaData(engine)
# 第二种方式
article_table = Table('article', metadata,
                      Column('title', String(30), primary_key=True),
                      Column('content', Text),
                      )
# 创建
metadata.create_all()
```

----
### 创建一个会话

----
```
from sqlalchemy.orm import sessionmaker
Sessoin = sessionmaker(bind=engine)
sessoin = Sessoin()
```
----
### 添加操作
-----
#### 添加一条数据，提交
```
user_1 = User(username='user_1', password='user_1')
# sessoin.add(user_1)
```
#### 添加多条数据，提交
```
user_2 = User(username='user_2', password='user_2')
user_3 = User(username='user_3', password='user_3')
# sessoin.add_all([user_2, user_3])
# sessoin.commit()
```
---
### 2.查找操作
----
#### 查找某个表中的全部数据
```
select_all = sessoin.query(User).all()
print(select_all)
```
#### 查找某个表中的第一条数据，某一列
```
# 查找表中的第一条数据
select_first = sessoin.query(User).first()
# 查找第一条数据的username那一列
select_first_username=sessoin.query(User.username).first()
print(select_first)
```
#### 使用filter,filter_by查找的时候筛选数据
```
# 查找的时候筛选数据,配合all 或者 first
select_filterby = sessoin.query(User.password).filter_by(username='user_1').all()
# print(select_filterby)
# filter和filter_by很像，前者用于简单查询，注意到使用filter查询时，需要添加对象名，要不然会报错，
# 后者用于复杂查询
# equals
select_filter_eq = sessoin.query(User).filter(User.username == 'user_1').all()
# not equals
select_filter_nq = sessoin.query(User).filter(User.username != 'user_1')
# more than
select_filter_gt = sessoin.query(User).filter(User.id > 5).all()
# less than
select_filter_lt = sessoin.query(User).filter(User.id < 5).all()
# like
select_filter_like = sessoin.query(User).filter(User.username.like('u%'))
# in
select_filter_in = sessoin.query(User).filter(User.username.in_(['user_1', 'user_2']))
print(select_filter_in)
# not in
 select_filter_not_in = sessoin.query(User).filter(~User.id.in_([2, 3]))
print(select_filter_not_in)
# print(select_filter_eq)
# is null
select_filter_null = sessoin.query(User).filter(User.username._is_(None))
# and
from sqlalchemy import and_

sessoin.query(User).filter(and_(User.id == 1, User.username == 'user_1'))
sessoin.query(User).filter(User.id == 1).filter(User.username == 'user_1')

# or
from sqlalchemy import or_

sessoin.query(User).filter(or_(User.id == 1, User.username == 'user_1'))
# match
sessoin.query(User).filter(User.username.match('user_1'))
```
#### 使用text，配合filter,filter_by
```
 # 使用text,配合filter,filter_by
from sqlalchemy import text
 sessoin.query(User).filter(text("id>2")).order_by(text('id')).all()
# 使用params，添加参数
 sessoin.query(User).filter(text('id >2 and  name=:name').params(name='user_1'))
```
#### 使用 statement,配合text执行原生的sql语句
```
# 使用from_statement执行原生的sql语句
sessoin.query(User).from_statement(text("select * from users"))
# 同样使用params，添加参数
sessoin.query(User).from_statement(text("select * from users where name=:name").params(name='user_1'))
```
#### 使用count统计数量
```
# 统计username='user_1'的数量
b = sessoin.query(User).filter(User.username == 'user_1').count()
print(b)
# 统计某些组的个数，使用func.count()
# [(1, 'Brian Johnson'), (1, 'Charles Barber'), (1, 'Kathryn Walker'), (1, 'Margaret
# Fox'), (1, 'Marie Reyes'), (1, 'Michael Nichols'), (1, 'Mrs. Karen Delacruz'), (1, 'Randy Wallace'), (1,
# 'Teresa Cochran'), (1, 'Tyler Smith'), (2, 'user_1'), (3, 'user_2'), (4, 'user_3')]
from sqlalchemy import func
print(sessoin.query(func.count(User.username), User.username).group_by(User.username).all())
# 如果统计有多少行，需要用select_from 选定一个表
d = sessoin.query(func.count('*')).select_from(User).scalar()
# d ==19
print(d)
# 统计某一列的数量，则不需要
print(sessoin.query(func.count(User.username)).scalar())
```
#### 关联查询,join
```
# 关联查询
# User 和article是一对多的关系
# 这里是差找每一个user所拥有的article
for u, a in sessoin.query(User, Article).filter(User.id == Article.user_id).all():
print(u, a.title)
# 也可以使用join
sessoin.query(User).join(Article).filter(User.id == Article.user_id).all()
# 搜索有发表article的user。
sessoin.query(User).join(Article, Article.user_id == User.id).all()
```
----
### 3.删除操作
----
```
# 删除数据前，先找到具体要删除的数据
delete_data = sessoin.query(User).filter_by(username='user_1').first()
sessoin.delete(delete_data)
#sessoin.commit()
```
-----
### 4.修改数据
----
```
# 需要数据一般是把数据找出来，在修改字段提交
select_filterby = sessoin.query(User).filter_by(username='user_1').first()
select_filterby.password = '123'
sessoin.commit()
```

完整代码
```
from sqlalchemy import create_engine, Text, ForeignKey, Column, String, Integer, Table, MetaData
from faker import Factory
# 连接，创建

# sqlite3
from sqlalchemy.orm import relationship, sessionmaker

engine = create_engine('sqlite:///data.sqlite')
print(engine)

# 使用orm
from sqlalchemy.ext.declarative import declarative_base

"""
 ----1---- 
"""
# 基类
Base = declarative_base()


# 设计表的结构
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String(64), nullable=False)
    password = Column(String(64), nullable=False, index=True)
    articles = relationship('Article', backref='author')
    # 一对一关系，加上userlist=False
    userinfo = relationship('UserInfo', backref='user', uselist=False)

    def __repr__(self):
        return f'{self.username}'


class UserInfo(Base):
    __tablename__ = 'userinfos'
    id = Column(Integer, primary_key=True)
    name = Column(String(64))
    qq = Column(String(11))
    phone = Column(String(11))
    link = Column(String(64))
    user_id = Column(Integer, ForeignKey('users.id'))


class Article(Base):
    __tablename__ = 'articles'

    id = Column(Integer, primary_key=True)
    title = Column(String(255), nullable=False, index=True)
    content = Column(Text)
    user_id = Column(Integer, ForeignKey('users.id'))
    tags = relationship('Tag', secondary='article_tag', backref='articles')

    # author = relationship('User')

    def __repr__(self):
        return '%s' % self.__class__.__name__


class Tag(Base):
    __tablename__ = 'tags'

    id = Column(Integer, primary_key=True)
    name = Column(String(64), nullable=False, index=True)

    def __repr__(self):
        return self.__class__.__name__


# 多对多关系，需要分解成两个一对多关系。
article_tag = Table(
    'article_tag', Base.metadata,
    Column('article_id', Integer, ForeignKey('articles.id')),
    Column('tag_id', Integer, ForeignKey('tags.id'))
)

"""
----2----
"""

# 绑定引擎
# metadata = MetaData(engine)
# 删除表
# metadata.drop_all()
# 设置表的结构
"""
user_table = Table('users', metadata,
                   Column('id', Integer, primary_key=True),
                   Column('username', String(64), nullable=False),
                   Column('password', String(64), nullable=False, index=True),
                   )
articles = relationship('article_table')
article_table = Table('articles', metadata,
                      Column('title', String(255), primary_key=True),
                      Column("content", Text),
                      Column('user_id', Integer, ForeignKey('users.id')),
                      )
author = relationship('user_table')
"""

# 创建表
# metadata.create_all()

# 连接表
# autoload_with：指定引擎
# u = Table('user', metadata, autoload=True)
# print(dir(u))


if __name__ == '__main__':
    # 创建表
    # Base.metadata.create_all(engine)
    # 创建一个会话
    # faker = Factory.create()
    Sessoin = sessionmaker(bind=engine)
    sessoin = Sessoin()
    # faker_users = [
    #    User(username=faker.name(),
    #         password=faker.word(),
    #         ) for i in range(10)]
    # sessoin.add_all(faker_users)

    # faker_tags = [Tag(name=faker.word()) for i in range(5)]
    # sessoin.add_all(faker_tags)

    # for i in range(100):
    #    article = Article(
    #        title=faker.sentence(),
    #        content=faker.sentence(),
    #        author=random.choice(faker_users)
    #    )
    #    for tag in random.sample(faker_tags, random.randint(2, 5)):
    #        article.tags.append(tag)
    #    sessoin.add(article)
    # sessoin.commit()

    # 添加一条操作
    # user_1 = User(username='user_1', password='user_1')
    # sessoin.add(user_1)
    # 添加多条数据
    # user_2 = User(username='user_2', password='user_2')
    # user_3 = User(username='user_3', password='user_3')
    # sessoin.add_all([user_2, user_3])
    # sessoin.commit()
    # 查找某个表中的全部数据
    select_all = sessoin.query(User).all()
    # print(select_all)
    # 查找表中的一条数据
    select_first = sessoin.query(User).first()
    # print(select_first)
    # 查找的时候筛选数据,配合all 或者 first
    select_filterby = sessoin.query(User.password).filter_by(username='user_1')
    # print(select_filterby)
    # filter和filter_by很像，前者用于简单查询，注意到使用filter查询时，需要添加对象名，要不然会报错，
    # 后者用于复杂查询
    # equals
    select_filter_eq = sessoin.query(User).filter(User.username == 'user_1')
    # not equals
    select_filter_nq = sessoin.query(User).filter(User.username != 'user_1')
    # more than
    select_filter_gt = sessoin.query(User).filter(User.id > 5).all()
    # less than
    select_filter_lt = sessoin.query(User).filter(User.id < 5).all()
    # like
    select_filter_like = sessoin.query(User).filter(User.username.like('u%'))
    # in
    select_filter_in = sessoin.query(User).filter(User.username.in_(['user_1', 'user_2']))
    # not in
    select_filter_not_in = sessoin.query(User).filter(~User.id.in_([2, 3]))
    from sqlalchemy.orm.exc import MultipleResultsFound
    # print(sessoin.query(User.id).filter(User.username == 'user_1').order_by(User.id).one())

    # print(select_filter_eq)
    # and
    from sqlalchemy import and_

    sessoin.query(User).filter(and_(User.id == 1, User.username == 'user_1'))
    sessoin.query(User).filter(User.id == 1).filter(User.username == 'user_1')

    # or
    from sqlalchemy import or_

    sessoin.query(User).filter(or_(User.id == 1, User.username == 'user_1'))
    # match
    sessoin.query(User).filter(User.username.match('user_1'))

    # 使用text,配合filter,filter_by
    from sqlalchemy import text

    sessoin.query(User).filter(text("id>2")).order_by(text('id')).all()
    # 使用params，添加参数
    sessoin.query(User).filter(text('id >2 and  name=:name').params(name='user_1'))
    # 使用from_statement执行原生的sql语句
    sessoin.query(User).from_statement(text("select * from users"))
    # 同样使用params，添加参数
    sessoin.query(User).from_statement(text("select * from users where name=:name").params(name='user_1'))

    # 统计username='user_1'的数量
    b = sessoin.query(User).filter(User.username == 'user_1').count()
    print(b)
    # 统计某些组的个数，使用func.count()
    # [(1, 'Brian Johnson'), (1, 'Charles Barber'), (1, 'Kathryn Walker'), (1, 'Margaret
    # Fox'), (1, 'Marie Reyes'), (1, 'Michael Nichols'), (1, 'Mrs. Karen Delacruz'), (1, 'Randy Wallace'), (1,
    # 'Teresa Cochran'), (1, 'Tyler Smith'), (2, 'user_1'), (3, 'user_2'), (4, 'user_3')]
    from sqlalchemy import func

    print(sessoin.query(func.count(User.username), User.username).group_by(User.username).all())
    # 如果统计有多少行，需要用select_from 选定一个表
    d = sessoin.query(func.count('*')).select_from(User).scalar()
    # d ==19
    print(d)
    # 统计某一列的数量，则不需要
    print(sessoin.query(func.count(User.username)).scalar())

    # 关联查询
    # User 和article是一对多的关系
    # 这里是差找每一个user所拥有的article
    for u, a in sessoin.query(User, Article).filter(User.id == Article.user_id).all():
        print(u, a.title)
    # 也可以使用join
    sessoin.query(User).join(Article).filter(User.id == Article.user_id).all()
    # 搜索有发表article的user。
    sessoin.query(User).join(Article, Article.user_id == User.id).all()
    # 删除数据前，先找到具体要删除的数据
    # delete_data = sessoin.query(User).filter_by(username='user_1').first()
    # sessoin.delete(delete_data)
    # sessoin.commit()

    # 需要数据一般是把数据找出来，在修改字段提交
    # select_filterby = sessoin.query(User).filter_by(username='user_1').first()
    # select_filterby.password = '123'
    # sessoin.commit()
```