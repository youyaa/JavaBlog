## 并发事务

- 读-读情况：允许。

- 写-写情况：可能会出现脏写，所以必须通过加锁方式来防止这种情况的发生。

- 读-写或写-读情况：也就是一个事务进行读取操作，另一个进行改动操作。

  这种情况可能发生脏读，不可重复读和幻读问题。解决方式是

  方案一：读操作利用多版本并发控制（MVCC），写操作进行加锁。

  > 所谓的`MVCC`我们在前一章有过详细的描述，就是通过生成一个`ReadView`，然后通过`ReadView`找到符合条件的记录版本（历史版本是由`undo日志`构建的），其实就像是在生成`ReadView`的那个时刻做了一次时间静止（就像用相机拍了一个快照），查询语句只能读到在生成`ReadView`之前已提交事务所做的更改，在生成`ReadView`之前未提交的事务或者之后才开启的事务所做的更改是看不到的。而写操作肯定针对的是最新版本的记录，读记录的历史版本和改动记录的最新版本本身并不冲突，也就是采用`MVCC`时，`读-写`操作并不冲突。

  方案二：读、写操作都采用`加锁`的方式。

  > 如果一些业务场景不允许读取记录的旧版本，而是每次都必须去读取记录的最新版本，比方在银行存款的事务中，你需要先把账户的余额读出来，然后将其加上本次存款的数额，最后再写到数据库中。在将账户余额读取出来后，就不想让别的事务再访问该余额，直到本次存款事务执行完成，其他事务才可以访问账户的余额。这样在读取记录的时候也就需要对其进行`加锁`操作，这样也就意味着`读`操作和`写`操作也像`写-写`操作那样排队执行。

很明显，采用`MVCC`方式的话，`读-写`操作彼此并不冲突，性能更高，采用`加锁`方式的话，`读-写`操作彼此需要排队执行，影响性能。一般情况下我们当然愿意采用`MVCC`来解决`读-写`操作并发执行的问题，但是业务在某些特殊情况下，要求必须采用`加锁`的方式执行。

## 一致性读

事务利用`MVCC`进行的读取操作称之为`一致性读`，或者`一致性无锁读`，有的地方也称之为`快照读`。所有普通的`SELECT`语句（`plain SELECT`）在`READ COMMITTED`、`REPEATABLE READ`隔离级别下都算是`一致性读`，比方说：

```
SELECT * FROM t;
SELECT * FROM t1 INNER JOIN t2 ON t1.col1 = t2.col2
```

`一致性读`并不会对表中的任何记录做`加锁`操作，其他事务可以自由的对表中的记录做改动。

## 锁定读

- `共享锁`，英文名：`Shared Locks`，简称`S锁`。在事务要读取一条记录时，需要先获取该记录的`S锁`。
- `独占锁`，也常称`排他锁`，英文名：`Exclusive Locks`，简称`X锁`。在事务要改动一条记录时，需要先获取该记录的`X锁`。

### 锁定读的语句

- 对读取的记录加`S锁`：

  ```
  SELECT ... LOCK IN SHARE MODE;
  ```

  也就是在普通的`SELECT`语句后边加`LOCK IN SHARE MODE`，如果当前事务执行了该语句，那么它会为读取到的记录加`S锁`，这样允许别的事务继续获取这些记录的`S锁`（比方说别的事务也使用`SELECT ... LOCK IN SHARE MODE`语句来读取这些记录），但是不能获取这些记录的`X锁`（比方说使用`SELECT ... FOR UPDATE`语句来读取这些记录，或者直接修改这些记录）。如果别的事务想要获取这些记录的`X锁`，那么它们会阻塞，直到当前事务提交之后将这些记录上的`S锁`释放掉。

- 对读取的记录加`X锁`：

  ```
  SELECT ... FOR UPDATE;
  ```

  也就是在普通的`SELECT`语句后边加`FOR UPDATE`，如果当前事务执行了该语句，那么它会为读取到的记录加`X锁`，这样既不允许别的事务获取这些记录的`S锁`（比方说别的事务使用`SELECT ... LOCK IN SHARE MODE`语句来读取这些记录），也不允许获取这些记录的`X锁`（比如说使用`SELECT ... FOR UPDATE`语句来读取这些记录，或者直接修改这些记录）。如果别的事务想要获取这些记录的`S锁`或者`X锁`，那么它们会阻塞，直到当前事务提交之后将这些记录上的`X锁`释放掉。

## 多粒度锁

前边提到的`锁`都是针对记录的，也可以被称之为`行级锁`或者`行锁`，对一条记录加锁影响的也只是这条记录而已，我们就说这个锁的粒度比较细；其实一个事务也可以在`表`级别进行加锁，自然就被称之为`表级锁`或者`表锁`，对一个表加锁影响整个表中的记录，我们就说这个锁的粒度比较粗。给表加的锁也可以分为`共享锁`（`S锁`）和`独占锁`（`X锁`）

- 意向共享锁，英文名：`Intention Shared Lock`，简称`IS锁`。当事务准备在某条记录上加`S锁`时，需要先在表级别加一个`IS锁`。
- 意向独占锁，英文名：`Intention Exclusive Lock`，简称`IX锁`。当事务准备在某条记录上加`X锁`时，需要先在表级别加一个`IX锁`。

总结一下：IS、IX锁是表级锁，它们的提出仅仅为了在之后加表级别的S锁和X锁时可以快速判断表中的记录是否被上锁，以避免用遍历的方式来查看表中有没有上锁的记录，也就是说其实IS锁和IX锁是兼容的，IX锁和IX锁是兼容的。

## MySQL中的行级锁

- Record Locks：

  我们前边提到的记录锁就是这种类型，也就是仅仅把一条记录锁上，我决定给这种类型的锁起一个比较不正经的名字：`正经记录锁`（请允许我皮一下，我实在不知道该叫个啥名好）。官方的类型名称为：`LOCK_REC_NOT_GAP`。

  `正经记录锁`是有`S锁`和`X锁`之分的，让我们分别称之为`S型正经记录锁`和`X型正经记录锁`吧（听起来有点怪怪的），当一个事务获取了一条记录的`S型正经记录锁`后，其他事务也可以继续获取该记录的`S型正经记录锁`，但不可以继续获取`X型正经记录锁`；当一个事务获取了一条记录的`X型正经记录锁`后，其他事务既不可以继续获取该记录的`S型正经记录锁`，也不可以继续获取`X型正经记录锁`。

- Gap Locks：

  我们说`MySQL`在`REPEATABLE READ`隔离级别下是可以解决幻读问题的，解决方案有两种，可以使用`MVCC`方案解决，也可以采用`加锁`方案解决。但是在使用`加锁`方案解决时有个大问题，就是事务在第一次执行读取操作时，那些幻影记录尚不存在，我们无法给这些幻影记录加上`正经记录锁`。不过这难不倒设计`InnoDB`的大叔，他们提出了一种称之为`Gap Locks`的锁，官方的类型名称为：`LOCK_GAP`，我们也可以简称为`gap锁`。

