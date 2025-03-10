---
title: 设备监控面板和告警（influxdb+telegraph+grafana+kapacitor）
date: 2024-10-21 17:38:09
tags:
categories: [Program, Web]
---

# 一、前言

服务端开发运维需要有一个平台和页面能看到设备的状态信息，如cpu、内存、磁盘、网络等。这些使用influxdb+telegraph+grafana可以很容易实现，本文从搭建使用介绍这三款软件如何配合达成此场景。

kapacitor

# 二、知识介绍

<img src="2024-10-21-01.png" />

- telegraph是一个收集信息的组件，将信息存到influxdb中
- influxdb是一个时序数据库
- grafana是一个将influxdb中的数据展示出来的

# 三、组件命令详解

## 1. influxdb 流数据库

### 1.1. 常用命令

```shell
########## 数据库操作 ##########
# 创建数据库
CREATE DATABASE <database_name>
# 删除数据库
DROP DATABASE <database_name>
# 查看所有数据库
SHOW DATABASES
# 使用数据库
USE <database_name>

########## 表（measurement）操作 ##########
# 插入数据
INSERT INTO <measurement_name> (field_key1, field_key2) VALUES (value1, value2)
# 查看所有表
SHOW MEASUREMENTS
# 数据查询，类似sql，但支持时间
SELECT * FROM <measurement_name> WHERE time >= '2024-01-01T00:00:00Z' AND time <= '2024-01-31T23:59:59Z'
# 聚合查询，支持mean、min、max
SELECT MEAN(<field_key>) FROM <measurement_name> WHERE time >= now() - 1h
# 删除数据
DELETE FROM <measurement_name> WHERE <condition>
# 删除表
DROP MEASUREMENT <measurement_name>
# 查看表结构
SHOW FIELD KEYS FROM <measurement_name>
# influxdb添加列可以直接插入新列即可，但不支持删除列，只能新建表放入
INSERT INTO <measurement_name> (time, "field1", "field2", "new_field") VALUES (now(), 10, 20, 30)

########## 用户 ##########
# 查看所有用户
SHOW USERS
# 创建用户
CREATE USER <username> WITH PASSWORD '<password>'
# 授予权限
GRANT ALL ON <database_name> TO <username>
```

### 1.2. 时间格式化显示

需要输入`precision rfc3339`，然后输出就会按照RTC显示，在语句后面加上`tz('Asia/Shanghai')`转化成我们的时区
