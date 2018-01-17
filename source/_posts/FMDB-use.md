---
title: FMDB使用
date: 2017-04-11 20:57:24
tags:
---

# FMDB

FMDB的下载地址 <https://github.com/ccgus/fmdb>

FMDB的安装  
FMDB can be installed using CocoaPods.

	pod 'FMDB'
	# pod 'FMDB/FTS'   # FMDB with FTS
	# pod 'FMDB/standalone'   # FMDB with latest SQLite amalgamation source
	# pod 'FMDB/standalone/FTS'   # FMDB with latest SQLite amalgamation source and FTS
	# pod 'FMDB/SQLCipher'   # FMDB with SQLCipher

## USage
There are three main classes in FMDB:

1. `FMDatabase` - Represents a single SQLite database. Used for executing SQL statements.
2. `FMResultSet` - Represents the results of executing a query on an FMDatabase.
3. `FMDatabaseQueue` - If you're wanting to perform queries and updates on multiple threads, you'll want to use this class. It's described in the "Thread Safety" section below.

## 注意
 1. 在多线程中推荐使用`FMDatabaseQueue`，为了保证线程安全。
 2. 为了提高执行效率，使用`Transactions`(事物)，批量执行sql语句。
 3. 同一个sqlite数据库支持同时在多个线程里面执行查询操作，但不支持同时执行更新和查询操作。

## 案例
#### 案例一
当正在更新数据库的时候，用户执行了查询操作
假如更新数据库需要2分钟，那么就会出现页面卡顿的情况，当数据库更新完毕后，才会执行用户的查询操作。

解决方法：

1. 更新数据库使用`Transactions`（事务），一般情况2万条数据需要3s左右，还是挺快的。
2. 但有时候设备性能比较差，可能需要40s左右，这样会让用户等待40s。
使用`Transactions`（事务）批量化执行sql语句，使用延迟事务操作。

	如2万条数据，可以选择每次更新后，当前分线程休眠2s（足够用户执行查询操作），然后继续执行事务操作，以此类推，直到数据库更新完毕。
