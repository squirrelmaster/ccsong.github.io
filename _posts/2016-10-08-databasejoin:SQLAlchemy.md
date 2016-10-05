---
layout: post
title: 数据库连接：SQLAlchemy"
date: 2016-10-08
categories: blog
tags: [database]

---


### SQLAlchemy简介
Python包SQLAlchemy整合了从建表，数据库存取，查询，修改这样整套流畅的生产线，大大提高了使用效率，将SQL语句封装在包中，直接调用即可。
在写爬虫程序抓取实时消息时，可以提前封装好需要的数据库建表包，和定时任务结合爬取，实现自动定时爬取及时存档数据库，更新数据的功能。

#### Database Urls
- dialect+driver://username:password@host:port/database

<center>
    <p><img src="https://raw.githubusercontent.com/squirrelmaster/squirrelmaster.github.io/master/img/sqla_engine_arch.png" align="center"></p>
</center>


- Dialects
  + # default
  + engine = create_engine('postgresql://scott:tiger@localhost/mydatabase')

  + # psycopg2
  + engine = create_engine('postgresql+psycopg2://scott:tiger@localhost/mydatabase')

  + # pg8000
  + engine = create_engine('postgresql+pg8000://scott:tiger@localhost/mydatabase')


### 安装
$ easy_install sqlalchemy

- [使用SQLAlchemy](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0014021031294178f993c85204e4d1b81ab032070641ce5000)

