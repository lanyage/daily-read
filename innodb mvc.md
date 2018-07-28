# innodb mvcc
为了通过无锁的方式实现可重复读，mysql的innodb引擎支持mvcc(多版本并发控制)，当然不同的数据库mvcc有不同的实现。

之所以能够实现无锁的可重复读，原理如下。

对于select，只有满足以下两个条件的记录才会被查出来。

1. select只会查询小于或等于当前事物ID的行，因为这样能保证select的数据是事物之前开始就存在的或事物自身插入或修改过的。
2. 行的删除版本要么是undefined,要么大于当前事物版本。这可以确保事物读取的行，在事物开始之前未被删除。


innodb给每行增加了两个字段,一个是行被插入的事物的id,另一个是行被删除或者更新的事物的id。

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|undefined|
|2|lisi|1|undefined|


事务a执行的时候:

	start transaction;
	select * from test;
	select * from test;
	commit;
	
## 场景1
若在事物a还没执行到第2个select的时候,事物b插入了一条数据

	start transaction;
	insert into test values(null, wangwu);
	commit;

那么数据库现在的是这样的:

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|undefined|
|2|lisi|1|undefined|
|3|wangwu|3|undefined|

因为3>a的事物的id:2，所以事物a查出来的记录还是:

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|undefined|
|2|lisi|1|undefined|


## 场景2
若在事物a还没执行到第2个select的时候,c删除了一条数据。
	
	start transaction;
	delete from test where id = 1;
	commit;

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|4|
|2|lisi|1|undefined|
|3|wangwu|3|undefined|

因为4>a的事物的id:2,因此a查出来的数据还是:

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|undefined|
|2|lisi|1|undefined|

## 场景3
若在事物a还没执行到第2个select的时候,d更新了一条数据,实际上是插入了一条数据.
	
	start transaction;
	update test set name = zhaoliu where id = 2;
	commit;

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|4|
|2|lisi|1|5|
|3|wangwu|3|undefined|
|2|lisi|5|undefined|

这样a读出来的数据还是:

|id|name|create_tx_id|delete_tx_id|
|:---:|:---:|:---:|:---:|
|1|zhangsan|1|undefined|
|2|lisi|1|undefined|
