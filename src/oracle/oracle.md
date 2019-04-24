# Oracle

## 1、常用函数



| str函数                                     |                                                            |
| ------------------------------------------- | ---------------------------------------------------------- |
| concat                                      | 连接字符串                                                 |
| length(x)                                   | 返回 x的长度                                               |
| lower（x）                                  | x转为大写                                                  |
| upper（x）                                  | x转为小写                                                  |
| initcap（x）                                | 首字母大写                                                 |
| ltrim（x，[str]）                           | 把X的左边截去str字符串，缺省截去空格                       |
| rtrim（x，[str]）                           | 把X的右边截去str字符串，缺省截去空格                       |
| trim（x，[str]）                            | 把X的两边截去str字符串，缺省截去空格                       |
| replace（x，old，new）                      | 在x中查找old，替换为new                                    |
| substr（x，start，length）                  | 在x中从start开始截取length长度的字符                       |
| **日期函数**                                |                                                            |
| last_day                                    | 本月最后一天 select last_day(sysdate) from dual;           |
| add_months(d,n)                             | 当前日期d后推n个月 select add_months(sysdate,2) from dual; |
| to_char(sysdate,'**MM**')                   | 月份数                                                     |
| to_char(sysdate,'**ww**')                   | 当年第几周                                                 |
| to_char(sysdate,'**w**')                    | 本月第几周                                                 |
| to_char(sysdate,'**DDD**')                  | 当年第几天                                                 |
| to_char(sysdate,'**DD**')                   | 当月第几天                                                 |
| to_char(sysdate,'**D**')                    | 周内第几天                                                 |
| to_char(sysdate,'**DY**')                   | 周内第几天缩写                                             |
| to_date('str_date','yyyy-mm-dd'),'**DAY**') | 某天是星期几                                               |
| to_char(sysdate,'**hh12**')                 | 12小时制小时数                                             |
| to_char(sysdate,'**hh24**')                 | 24小时制小时数                                             |
| to_char(sysdate,'**Mi**')                   | 分钟数                                                     |
| to_char(sysdate,'**ss**')                   | 秒数                                                       |
| **month_between**(sysdate,'date')           | 日期相差的月份                                             |
| add_month(sysdate,5)                        | 日期加上指定月份                                           |
| next_day(sysdate,5)                         | 指定日期的下一个周几                                       |
| **to_date**('2004-11-27','yyyy-mm-dd')      | str的日期变为date类型                                      |
| union                                       | 俩个查询结果并集                                           |
| intersect                                   | 俩个查询结果交集                                           |
| minus                                       | 俩个查询结果差集                                           |

## 2、常用操作

### 	2.1 导入导出表

导入表

```shell
imp username/pwd@localhost/ORCL file=".dmp文件路径" full=y ignore=y
```

### 	2.2 密码过期

```sql
SELECT username,profile FROM dba_users
SELECT * FROM dba_profiles s WHERE s.profile='DEFAULT' AND resource_name='PASSWORD_LIFE_TIME';
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

### 	2.3 修改密码

```sql
alter user scott identified by tiger;
```

### 	2.4 创建表

```sql
create table course (
    cno number(3) primary key ,
    cname varchar2(10)
);
```

​	需要注意的点 

​	（1）、如果是int类型的字段，最好使用number（字段大小）类型，如果要使用int和integer类型 后边不能加大小。

​	（2）、每个字段后边是，号

​	（3）、创建表的最后一个字段没有，号，有了会报错