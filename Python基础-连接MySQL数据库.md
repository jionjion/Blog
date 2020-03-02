---
title: Python基础-连接MySQL数据库
categories:
  - Python
tags:
  - Python
  - MySQL
abbrlink: c00211ab
date: 2020-02-26 19:39:40
---



## ~~通过 `MySQLdb` 连接~~

安装 `MySQLdb` 模块
`pip install MySQLdb`

[官网地址](https://mysqlclient.readthedocs.io/user_guide.html)



### 数据库表结构

| id   | name           | author          | publication_date    |
| ---- | -------------- | --------------- | ------------------- |
| 1    | 冰与火之歌     | 乔治·马丁       | 2020-02-26 12:32:14 |
| 2    | 三体           | 刘慈宁          | 2020-02-26 13:11:11 |
| 3    | 银河帝国三部曲 | 艾萨克·阿西莫夫 | 1951-01-01 00:00:00 |



###创建连接

通过 `MySQLdb.connect` 创建连接.注意连接需要在使用后关闭连接
`MySQLdb.Error` 为模块抛出异常,所有的异常均被此定义

```python
import MySQLdb

class MysqlConnect:
    """
        连接MySQL数据库
    """

    # 创建数据库连接
    def get_connect(self):
        try:
            conn = MySQLdb.connect(
                host='127.0.0.1',
                user='root',
                passwd='123456',
                db='python',
                port=3306,
                charset='utf8'
            )
            return conn

        # 异常处理
        except MySQLdb.Error as e:
            # 打印异常
            print('Error: (%s)' % (e,))

    # 关闭连接
    def close_connect(self, conn):
        conn.close()


class MysqlConnectTest:
    """
        测试连接数据库类
    """
    if __name__ == '__main__':
        obj = MysqlConnect()
        # 获得连接
        conn = obj.get_connect()
        # 关闭连接
        obj.close_connect(conn)
```



###　查询

通过连接对象创建 `cursor` 游标,所有操作均由游标操作
游标通过 `fetchone` 和 `fetchall` 方法,抓取一个或者多个结果
注意将结果转为 `JSON` 对象,并对外返回.
注意,在每次查询结束后,关闭游标和连接

```python
import MySQLdb
import datetime

class MysqlSelect:
    """
        查询语句
    """

    # 在类初始化时,创建连接绑定在对象参数中
    def __init__(self):
        # 调用获得连接方法
        try:
            # 绑定连接到对象参数中
            self.conn = MySQLdb.connect(
                host='127.0.0.1',
                user='root',
                passwd='123456',
                db='python',
                port=3306,
                charset='utf8'
            )
        except MySQLdb.Error as e:
            print('Error: (%s)' % (e,))

    # 关闭连接,如果连接存在
    def close_conn(self):
        try:
            if self.conn:
                self.conn.close()
        except MySQLdb.Error as e:
            print('Error: (%s)' % (e,))

    # 通过主键ID查询
    def get_one(self, book_id):
        # 准备SQL
        sql = 'select * from book where id = %s'
        # 生成游标
        cursor = self.conn.cursor()
        # 执行SQL,传入参数元组
        cursor.execute(query=sql, args=(book_id,))
        # 对结果处理,使用推导式,对记录进行键值对匹配
        result = dict(zip([k[0] for k in cursor.description], cursor.fetchone()))
        # 关闭游标连接,数据库连接
        cursor.close()
        # self.close_conn()
        # 返回结果
        return result

    # 字符条件查询,
    def get_column(self, book_name):
        # 通过反引号,标识这是一个表字段名或者表名
        sql = 'select * from `book` where `name` = %s'
        cursor = self.conn.cursor()
        cursor.execute(query=sql, args=(book_name,))
        result = [dict(zip([k[0] for k in cursor.description], row)) for row in cursor.fetchall()]
        cursor.close()
        # self.close_conn()
        return result

    # 模糊查询
    def get_like(self, book_name):
        sql = "select * from `book` where `name` like %s"
        cursor = self.conn.cursor()
        # 使用%%表示百分号,并被单引号包裹,进行模糊查询
        cursor.execute(query=sql, args=('%' + book_name + '%',))
        result = [dict(zip([k[0] for k in cursor.description], row)) for row in cursor.fetchall()]
        cursor.close()
        # self.close_conn()
        return result

    # 范围查询
    def get_in(self, id_list):
        sql = 'select * from `book` where `id` in %s'
        cursor = self.conn.cursor()
        # 传入一个列表元组
        cursor.execute(query=sql, args=(id_list,))
        result = [dict(zip([k[0] for k in cursor.description], row)) for row in cursor.fetchall()]
        cursor.close()
        # self.close_conn()
        return result

    # 日期查询
    def get_date(self, date_from, date_to):
        sql = "select * from `book` where `publication_date` between %s and %s"
        cursor = self.conn.cursor()
        # 日期查询直接将日期格式化后传入
        cursor.execute(query=sql, args=(date_from, date_to))
        result = [dict(zip([k[0] for k in cursor.description], row)) for row in cursor.fetchall()]
        cursor.close()
        # self.close_conn()
        return result

    # 分页查询
    def get_page(self, page, page_size):
        # 起始条数,和偏移量
        sql = "select * from `book` limit %s , %s"
        cursor = self.conn.cursor()
        # (当前页-1) * 每页数量 , 偏移量
        cursor.execute(query=sql, args=((page - 1) * page_size, page_size))
        result = [dict(zip([k[0] for k in cursor.description], row)) for row in cursor.fetchall()]
        cursor.close()
        # self.close_conn()
        return result


# 测试
class MysqlSelectTest:
    """
        测试查询
    """
    if __name__ == '__main__':
        obj = MysqlSelect()
        # 通过主键查询
        print("主键查询>> ", obj.get_one(1))
        # 通过字段查询
        print("字段查询>>", obj.get_column('三体'))
        # 通过模糊查询
        print("模糊查询>>", obj.get_like('三'))
        # 通过范围查询
        print("范围查询>>", obj.get_in([1, 2]))
        # 通过日期范围查询
        print("日期查询>>", obj.get_date('1000-01-01', '3000-01-01'))
        # 通过分页查询
        print("分页查询>>", obj.get_page(1, 1))
        # 关闭连接
        obj.close_conn()
```



###  增删改

游标通过 `commit()` 和 `rollback()` 进行提交事务和回滚事物

```python
import MySQLdb
import datetime

# 增删改
class MysqlDML:
    """
        增删改动作
    """

    # 在类初始化时,创建连接绑定在对象参数中
    def get_conn(self):
        # 调用获得连接方法
        try:
            # 绑定连接到对象参数中
            conn = MySQLdb.connect(
                host='127.0.0.1',
                user='root',
                passwd='123456',
                db='python',
                port=3306,
                charset='utf8'
            )
            return conn
        except MySQLdb.Error as e:
            print('Error: %s' % e)

    # 关闭连接,如果连接存在
    def close_conn(self, conn):
        try:
            if conn:
                conn.close()
        except MySQLdb.Error as e:
            print('Error: %s' % e)

    # 新增一条记录
    def add_one(self):
        conn = None
        cursor = None
        try:
            # 使用元组,将过于长的字符包裹,默认SQL参数只支持字符串类型
            sql = ("insert into `book`(`name`,`author`,`publication_date`) "
                   "value (%s, %s, %s)")
            # 获取数据库连接
            conn = self.get_conn()
            cursor = conn.cursor()
            # 获取当前时间
            now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            cursor.execute(query=sql, args=('冰与火之歌', '乔治·马丁', now))
            # 提交事务
            conn.commit()
        except MySQLdb.Error:
            # 一定要有异常捕获,避免事务部分失效,部分提交
            conn.rollback()
        finally:
            # 关闭连接
            cursor.close()
            self.close_conn(conn)

    # 新增多条记录
    def add_more(self):
        conn = None
        cursor = None
        try:
            # 使用元组,将过于长的字符包裹,默认SQL参数只支持字符串类型
            sql = ("insert into `book`(`name`,`author`,`publication_date`) "
                   "values (%s, %s, %s)")
            # 获取数据库连接
            conn = self.get_conn()
            cursor = conn.cursor()
            # 获取当前时间
            now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            # 批量数据,使用列表封装元组数据
            data = [('冰与火之歌', '乔治·马丁', now), ('三体', '刘慈宁', now)]
            cursor.executemany(query=sql, args=data)
            # 提交事务
            conn.commit()
        except MySQLdb.Error:
            # 一定要有异常捕获,避免事务部分失效,部分提交
            conn.rollback()
        finally:
            # 关闭连接
            cursor.close()
            self.close_conn(conn)

    # 修改一条记录
    def modify_one(self, book_id):
        conn = None
        cursor = None
        try:
            # 使用元组,将过于长的字符包裹,默认SQL参数只支持字符串类型
            sql = "update `book` set publication_date = %s where `id` = %s"
            # 获取数据库连接
            conn = self.get_conn()
            cursor = conn.cursor()
            # 获取当前时间
            now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            cursor.execute(query=sql, args=(now, book_id))
            # 提交事务
            conn.commit()
        except MySQLdb.Error:
            # 一定要有异常捕获,避免事务部分失效,部分提交
            conn.rollback()
        finally:
            # 关闭连接
            cursor.close()
            self.close_conn(conn)

    #  删除一条记录
    def delete_one(self, book_id):
        conn = None
        cursor = None
        try:
            # 使用元组,将过于长的字符包裹,默认SQL参数只支持字符串类型
            sql = "delete from `book` where `id` = %s"
            # 获取数据库连接
            conn = self.get_conn()
            cursor = conn.cursor()
            # 注意,当为一个元素时,添加逗号,构成一个元组
            cursor.execute(query=sql, args=(book_id,))
            # 提交事务
            conn.commit()
        except MySQLdb.Error:
            # 一定要有异常捕获,避免事务部分失效,部分提交
            conn.rollback()
        finally:
            # 关闭连接
            cursor.close()
            self.close_conn(conn)


# 增删改
class MysqlDMLTest:
    """
        测试增删改动作
    """
    if __name__ == '__main__':
        obj = MysqlDML()
        obj.add_one()
        obj.add_more()
        obj.modify_one(2)
        obj.delete_one(1)
```



## 通过 `pymysql` 连接



### 创建连接

```python
import pymysql

# 连接MySQL
class MysqlConnect:

    # 获取数据库连接
    def get_connect(self):
        conn = pymysql.Connect(host="127.0.0.1", user="root", password="123456",
                               database="python", port=3306, charset='utf8')
        return conn

    # 关闭连接
    def close_connect(self, conn):
        # 关闭连接
        conn.close()

    # 尝试获得数据
    def get_data(self):
        # 获得连接
        conn = self.get_connect()
        # 获得数据库游标,默认抓取的数据结构为元组类型
        cursor = conn.cursor()
        # SQL
        sql_select = "select * from `book`"
        # 执行查询,获得数据
        cursor.execute(sql_select)
        # 查看游标数据总数
        print("数据总数: %s" % (cursor.rowcount,))
        # 数据
        row_data = cursor.fetchall()
        print("数据: (%s)" % (row_data,))
        # 关闭游标
        cursor.close()
        # 关闭连接
        self.close_connect(conn)


# 测试连接MySQL
class MysqlConnectTest:
    if __name__ == '__main__':
        obj = MysqlConnect()
        obj.get_data()
```





### 查询

```python
import pymysql

# 查询
class MysqlSelect:
    # 获得连接
    def get_connect(self):
        conn = pymysql.Connect(host="127.0.0.1", user="root", password="123456",
                               database="python", port=3306, charset='utf8')
        return conn

    # 关闭连接
    def close_connect(self, conn):
        # 关闭连接
        conn.close()

    # 查询数据
    def get_rows(self):
        # 获得连接
        conn = self.get_connect()
        # 获得数据库游标,抓取的数据为JSON类型
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        # SQL
        sql_select = "select * from `book`"
        # 执行查询,获得数据
        cursor.execute(sql_select)
        # 抓取一条记录
        rs = cursor.fetchone()
        print("抓取一条记录:", rs)
        # 抓取指定记录
        rs = cursor.fetchmany(2)
        print("抓取两条记录", rs)
        # 移动游标
        # 相对当前位置移动,向后移动1位
        cursor.scroll(1, mode='relative')
        # 相对绝对位置移动,移动到第二个位置,随后向后移动2位
        cursor.scroll(2, mode='absolute')
        # 抓取全部记录
        rs = cursor.fetchall()
        # 对抓取的数据进行解析,仅限于元组数据结构
        print("抓取剩余记录", rs)
        for row in rs:
            print("用户名:", row[1])
        # 关闭游标
        cursor.close()
        # 关闭连接
        self.close_connect(conn)


# 测试查询
class MysqlSelectTest:
    if __name__ == '__main__':
        obj = MysqlSelect()
        obj.get_rows()
```





### 增删改

```python
import pymysql
import datetime

# 事物
class MysqlTransaction:
    # 获得连接
    def get_connect(self):
        conn = pymysql.Connect(host="127.0.0.1", user="root", password="123456",
                               database="python", port=3306, charset='utf8')
        return conn

    # 关闭连接
    def close_connect(self, conn):
        # 关闭连接
        conn.close()

    # 事物操作
    def modify(self):
        # 事务开启,默认自动提交事务是关闭的.如果不执行commit操作,数据的增删改是不会发生改变的
        # 获得连接
        conn = self.get_connect()
        # 获得数据库游标,抓取的数据为JSON类型
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        try:
            # 增加记录
            sql_insert = "insert into `book`(name, author, publication_date) values ('三体', '刘慈宁', '2020-02-29 00:00:00')"
            cursor.execute(sql_insert)
            # 修改记录
            sql_update = "update `book` set author = '刘慈宁' where id = 2"
            cursor.execute(sql_update)
            # 删除记录
            sql_delete = "delete from `book` where id = 7"
            cursor.execute(sql_delete)
            print("单条删除影响的记录数", cursor.rowcount)
            # 执行预编译的sql,并返回影响的行数,s%为占位符
            effect_row = cursor.execute("update `book` set author = '刘慈宁' where id = %s", (2,))
            print("预编译更新影响的记录数", effect_row)
            # 获取当前时间
            now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            data = [('冰与火之歌', '乔治·马丁', now), ('三体', '刘慈宁', now)]
            # 批量插入,传入list序列
            effect_row = cursor.executemany("insert into `book`(name, author, publication_date) values (%s, %s, %s)", data)
            print("批量插入影响的记录数", effect_row)
            # 获取自增ID,最后一条插入后的主键ID
            new_id = cursor.lastrowid
            print("最后一条记录主键ID:", new_id)
            # 事务提交
            conn.commit()
        except Exception as e:  # 如果出现异常,则回滚
            print("事务执行出现异常:", e)
            # 事务回滚
            conn.rollback()
        finally:
            # 关闭游标
            cursor.close()
            # 关闭连接
            conn.close()


# 测试事物
class MysqlTransactionTest:
    if __name__ == '__main__':
        obj = MysqlTransaction()
        obj.modify()
```







## 通过 `SQLAlchemy` 连接

更多数据类型: [参考](https://docs.sqlalchemy.org/en/13/core/type_basics.html#generic-types)



### 示例代码

```python
# -*- coding: utf-8 -*-

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import sessionmaker

import datetime

# 获取连接,指定数据库字符集
engine = create_engine('mysql+pymysql://root:123456@localhost:3306/python?charset=utf8mb4')

# 获取基类,数据模型类均继承该类
base = declarative_base()


# 声明数据模型,定义数据类型
class Book(base):
    # 基表
    __tablename__ = 'BOOK'

    # 数字字段类型,是否为主键
    id = Column(type_=Integer, primary_key=True)
    # 字符字段类型,且不为空
    name = Column(type_=String(200), nullable=False)
    author = Column(type_=String(200))
    # 日期类型
    publication_date = Column(type_=DateTime)

    # 重写描述方法
    def __repr__(self):
        return "<Book(id='%s', name='%s', author='%s', publication_date='%s' )>" \
               % (self.id, self.name, self.author, self.publication_date)


# 创建表,通过数据模型绑定数据库连接引擎
# Book.metadata.create_all(engine)

# 创建连接会话对象,并绑定数据库连接
Session = sessionmaker(bind=engine)


# 增删改查
class MysqlDML:
    # 初始化Session对象,作为类属性
    def __init__(self):
        self.session = Session()

    # 新增一条记录,并返回
    def add_one(self):
        # 创建对象
        book = Book(
            name='三体',
            author='刘慈宁',
            publication_date=datetime.datetime.now()
        )
        # 存入session中
        self.session.add(book)
        # 提交
        self.session.commit()
        return book

    # 批量提交
    def add_more(self):
        # 创建对象数组
        book1 = Book(
            name='银河帝国三部曲',
            author='艾萨克·阿西莫夫',
            publication_date=datetime.datetime.strptime('2001-01-01', '%Y-%m-%d')
        )
        book2 = Book(
            name='冰与火之歌',
            author='乔治·马丁',
            publication_date=datetime.datetime.strptime('1945-01-01', '%Y-%m-%d')
        )
        book_list = [book1, book2]
        # 存入,并提交事务
        self.session.add_all(book_list)
        self.session.commit()
        return book_list

    # 查询主键数据
    def get_one(self):
        # 获得主键为2的数据
        book = self.session.query(Book).get(2)
        return book

    # 条件查询
    def get_column(self):
        book_name = "三体"
        # 查询条件, 通过字段 name 过滤
        return self.session.query(Book).filter_by(name=book_name)

    # 模糊查询
    def get_like(self):
        return self.session.query(Book).filter(Book.name.like('%三%')).all()

    # 通过范围查询
    def get_in(self):
        return self.session.query(Book).filter(Book.id.in_([1, 2, 3])).all()

    # 通过日期范围查询
    def get_date(self):
        return self.session.query(Book).filter( datetime.datetime.strptime('2000-01-01', '%Y-%H-%d') < Book.publication_date , Book.publication_date <= datetime.datetime.now()).all()

    # 通过分页查询
    def get_page(self):
        return self.session.query(Book).limit(3).all()

    # 修改
    def modify_one(self):
        # 查询
        book = self.session.query(Book).get(2)
        # 修改内容,并保存
        book.name = '三体'
        self.session.add(book)
        # 提交事务
        self.session.commit()
        return book

    # 删除
    def delete_one(self):
        book = self.session.query(Book).get(18)
        if book:
            self.session.delete(book)
            self.session.commit()
            return book
        return None


# 测试
class MysqlDMLTest:
    """
        测试 ORM
    """
    if __name__ == '__main__':
        obj = MysqlDML()
        # 测试新增一条
        print(obj.add_one())
        # 测试批量提交
        print(obj.add_more())
        # 查询主键数据
        print(obj.get_one())
        # 测试条件查询
        print(obj.get_column())
        # 测试模糊查询
        print(obj.get_like())
        # 测试通过范围查询
        print(obj.get_in())
        # 测试通过日期范围查询
        print(obj.get_date())
        # 通过分页查询
        print(obj.get_page())
        # 测试修改
        print(obj.modify_one())
        # 测试删除
        print(obj.delete_one())

        # 通过点,调用内置方法. 查询属性,数量
        print(obj.get_one().name)
        print(obj.get_one().count())
```

