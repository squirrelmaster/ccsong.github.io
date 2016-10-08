---
layout: post
title: "数据库连接：SQLAlchemy高级用法【未完待续】"
date: 2016-10-09
categories: blog
tags: [database]

---


#### 高级应用
#### 建表
```
    from sqlalchemy import Column
    from sqlalchemy.types import CHAR, Integer, String  
    from sqlalchemy.ext.declarative import declarative_base   
    BaseModel = declarative_base()
    
    def init_db():
        BaseModel.metadata.create_all(engine)	
    def drop_db():
        BaseModel.metadata.drop_all(engine)
    class User(BaseModel):
          __tablename__ = 'user'
          id = Column(Integer, primary_key=True)
	   name = Column(CHAR(30)) # or Column(String(30))
    init_db()
```

#### 1.量插入大批数据
```
     session.execute(User.__table__.insert(),   
    [{'name': `randint(1, 100)`,'age': randint(1, 100)} for i in xrange(10000)])
    session.commit()
    上面批量插入了 10000 条记录，半秒内就执行完了；而 ORM 方式会花掉很长时间。
```

#### 2.如何让执行的 SQL 语句增加前缀？
```
    使用 query 对象的 prefix_with() 方法：
    session.query(User.name).prefix_with('HIGH_PRIORITY').all()
    session.execute(User.__table__.insert().prefix_with('IGNORE'), {'id': 1, 'name': '1'})
```

#### 3.如何替换一个已有主键的记录？
```
    使用 session.merge() 方法替代 session.add()，其实就是 SELECT + UPDATE：
    user = User(id=1, name='ooxx')
    session.merge(user)
    session.commit()
```
    
#### 4.如何使用无符号整数？
```
    可以使用 MySQL 的方言：
    from sqlalchemy.dialects.mysql import INTEGER
    id = Column(INTEGER(unsigned=True), primary_key=True)
```

#### 5.模型的属性名需要和表的字段名不一样
```
    有个其他系统的表里包含了一个“from”字段，这在 Python 里是关键字，于是只能这样处理了：
    from_ = Column('from', CHAR(10))
```

#### 6.如何获取字段的长度？
```
    Column 会生成一个很复杂的对象，想获取长度比较麻烦，这里以 User.name 为例：
    User.name.property.columns[0].type.length
```

#### 7.如何指定使用 InnoDB，以及使用 UTF-8 编码？
```
    最简单的方式就是修改数据库的默认配置。如果非要在代码里指定的话，可以这样：
    class User(BaseModel):
          __table_args__ = {
               'mysql_engine': 'InnoDB',
               'mysql_charset': 'utf8'
           }	   
    如果是对表来设置的话，可以把上面代码中的 utf8 改成 utf8mb4，DB_CONNECT_STRING 里的 charset 也
    这样更改。如果对库或字段来设置，则还是自己写SQL语句比较方便不建议全用utf8mb4代替utf8，因为前者更
    慢，索引会占用更多空间。
```
     
#### 8.如何设置外键约束？
```
    from random import randint
    from sqlalchemy import ForeignKey
    class User(BaseModel):
          __tablename__ = 'user'
          id = Column(Integer, primary_key=True)
          age = Column(Integer)
    class Friendship(BaseModel):
          __tablename__ = 'friendship'
          id = Column(Integer, primary_key=True)
          user_id1 = Column(Integer, ForeignKey('user.id'))
          user_id2 = Column(Integer, ForeignKey('user.id'))
	  
    for i in xrange(100):
       session.add(User(age=randint(1, 100)))
    session.flush() # 或 session.commit()，执行完后user对象的 id 属性才可以访问（因为id是自增的）
    
    for i in xrange(100):
       session.add(Friendship(user_id1=randint(1, 100), user_id2=randint(1, 100)))
    session.commit()
    session.query(User).filter(User.age < 50).delete()   
    【注】删除 user 表的数据，可能会导致 friendship 的外键不指向一个真实存在的记录。在默认情况下，
    MySQL 会拒绝这种操作，也就是 RESTRICT。InnoDB 还允许指定 ON DELETE 为 CASCADE 和 SET NULL，
    前者会删除 friendship 中无效的记录，后者会将这些记录的外键设为 NULL 除了删除，还有可能更改主键，
    这也会导致 friendship 的外键失效。于是相应的就有 ON UPDATE 了。其中 CASCADE 变成了更新相应的外
    键，而不是删除。而在 SQLAlchemy 中是这样处理的：
    class Friendship(BaseModel):
        __tablename__ = 'friendship'
        id = Column(Integer, primary_key=True)
        user_id1 = Column(Integer, ForeignKey('user.id', ondelete='CASCADE', onupdate='CASCADE'))
        user_id2 = Column(Integer, ForeignKey('user.id', ondelete='CASCADE', onupdate='CASCADE'))
    【补充】事件触发限制: on delete和on update , 可设参数cascade(跟随外键改动), restrict(限制外表中的
    外键改动),set Null(设空值）,set Default（设默认值）,[默认]no action
```

#### 9.正确使用事务
```
    MySQL InnoDB 虽然支持事务，但并不是那么简单的，还需要手动加锁。如果要保证事务运行期间内，被读取的数据不被修改，自己也不去修改，加读锁即可。如果需  	要更改数据，最好加写锁。进程锁的问题还在研究中，后续再行探究。
```
#### 未完待续......

#### 参考
- [databasejoin-SQLAlchemy](http://squirrelmaster.github.io/blog/2016/10/08/databasejoin-SQLAlchemy/)
 
