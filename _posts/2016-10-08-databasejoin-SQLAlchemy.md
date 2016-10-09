---
layout: post
title: "数据库连接：SQLAlchemy"
date: 2016-10-08
categories: blog
tags: [database]

---



### SQLAlchemy简介
Python包SQLAlchemy整合了从建表，数据库存取，查询，修改这样整套流畅的生产线，大大提高了使用效率，将SQL语句封装在包中，直接调用即可。
在写爬虫程序抓取实时消息时，可以提前封装好需要的数据库建表包，和定时任务结合爬取，实现自动定时爬取及时存档数据库，更新数据的功能。

#### Database Urls
- dialect+driver://username:password@host:port/database
- '数据库类型+数据库驱动名称://用户名:口令@机器地址:端口号/数据库名'

<center>
    <p><img src="https://raw.githubusercontent.com/squirrelmaster/squirrelmaster.github.io/master/img/sqla_engine_arch.png" align="center"></p>
</center>


#### Dialects
- postgresql
  + default： engine = create_engine('postgresql://scott:tiger@localhost/mydatabase')
  + psycopg2：engine = create_engine('postgresql+psycopg2://scott:tiger@localhost/mydatabase')
  + pg8000： engine = create_engine('postgresql+pg8000://scott:tiger@localhost/mydatabase')
- MySQL
  + default：engine = create_engine('mysql://scott:tiger@localhost/foo')
  + mysql-python：engine = create_engine('mysql+mysqldb://scott:tiger@localhost/foo')
  + MySQL-connector-python：engine = create_engine('mysql+mysqlconnector://scott:tiger@localhost/foo')
  + OurSQL：engine = create_engine('mysql+oursql://scott:tiger@localhost/foo')
- Oracle：The Oracle dialect uses cx_oracle as the default DBAPI
  + engine = create_engine('oracle://scott:tiger@127.0.0.1:1521/sidname')
  + engine = create_engine('oracle+cx_oracle://scott:tiger@tnsname')
- Microsoft SQL Server
  + pyodbc：engine = create_engine('mssql+pyodbc://scott:tiger@mydsn')
  + pymssql：engine = create_engine('mssql+pymssql://scott:tiger@hostname:port/dbname')

#### 安装
$ easy_install sqlalchemy

#### 导入基本包 
    from sqlalchemy import Column, Float, DateTime, String, Integer, Text, ForeignKey, UniqueConstraint, func
    from sqlalchemy.orm import sessionmaker
    from sqlalchemy import create_engine
    from sqlalchemy.dialects.mysql import LONGTEXT
    from sqlalchemy import text

#### Create Object Base
    from sqlalchemy.ext.declarative import declarative_base
    Base = declarative_base()
    
    class biao_name(Base):
    __tablename__ = 'yourtablename'
    CODE = Column(String(6), primary_key = True)
    TYPE = Column(String(10))
    TYPE_CODE = Column(Integer)
    RANGE = Column(Float)
    GGDATE = Column(DateTime, primary_key = True)
    BBDATE = Column(DateTime)
    UniqueConstraint('CODE', 'GGDATE')

    def __repr__(self):
        return "biao_name"
	
#### 接着封装建表：engine

    engine=create_engine('mysql+pymysql://%s:%s@%s/%s?charset=utf8mb4' %(user, password, host, database), echo = True)
    
    【注】utf8mb4包含了utf8的编码，但是占用空间大

    def createAll(self): #Create Table
        Base.metadata.create_all(self.engine)
    
    def makeSession(self): # SQL Session 建立
        #Create DB Session
        DBSession = sessionmaker(bind = self.engine)
        self.session = DBSession()
    
    def commit(self): #SQL Commit All Added Data
        self.session.commit()
    
    def close(self): #SQL Close the interface
        self.session.close()

    if __name__ == '__main__':
	    sql_handle = sqlio()
	    sql_handle.createAll()
	    sql_handle.makeSession()

#### 无法对表进行添加列操作，需要自行在sql中进行添加

#### 更多用法
- [SQLAlchemy使用经验](http://www.keakon.net/2012/12/03/SQLAlchemy使用经验)

#### 参考 
- [Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html)
- [使用SQLAlchemy](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0014021031294178f993c85204e4d1b81ab032070641ce5000)


