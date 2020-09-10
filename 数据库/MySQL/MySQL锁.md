表级锁：
分类一：**读锁**

```
lock table  student read;#读锁
可执行：select*from student;
等待解锁：当前不能执行insert，update，delete操作

unlock tables;#解锁
解锁后才可以执行insert 操作，update，delete
```

当我们执行锁表，使用read 读锁，则，在解锁之前，student表都只能读取，即只能执行select操作，插入操作insert 都处于等待解锁的状态。只能读，不能写。

**分类二：写锁**

```
lock table balance write; #写锁
insert into balance(`user_id`,`money`)values(7,77);
select * from `balance`;
#unlock TABLES; 
#insert into balance(`user_id`,`money`)values(6,44);
```

注意这里并没有解锁，但是成功插入了。
但是这是在一个mysql回话中插入的，如果是新的会话insert into语句还是会处于等待状态，等待这个表解锁。并且新的回话中也select语句也是如此。
意思是：如果是写锁没有解锁，新的 会话 读写 都处于被锁状态。
**行级锁**

myisam和innodb都支持表级锁。
行级锁只有innodb支持，也是mysql中的最小粒度锁，也是真正的事务锁。
**共享锁**

**行级锁分为：共享锁和排它锁**

```
**#共享锁**
select xxx lock in share mode;
```

这样就打开了共享锁，凡是select取出来的行数据只有该回话可以修改，直到commit之后其他回话才能修改，但是过程当中其他会话可以读取。

**行锁是索引级别的，并不是记录级别的**

```
start TRANSACTION;#开启事务

select * from `balance` where user_id=3 lock in share mode;

#commit;提交事务
```

在新的会话中，可以读取，但不能修改：

```
#另一个会话中

select * from `balance` where user_id=3;#可以读取

update balance set money=99 where user_id=3;#等待状态
```

这条语句是修改是user_id=3的条件，那么user_id=4的条件下能否修改？
发现也不能修改，这是为啥？
因为行锁是索引级别的，并不是记录级别的，我们的数据表user_id字段不是索引字段。

所以这告诉我们：在设置行级锁的时候，where条件里的字段应该是索引字段。

**排它锁**

```
select xxx for upadte;
```

打开了排它锁，其他会话依然不能修改数据。但是可以普通的读取。
如果另外一个会话也想加锁，则会冲突。