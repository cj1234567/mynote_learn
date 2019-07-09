# hive 集合数据类型

假设某表有如下一行，我们用JSON格式来表示其数据结构。在Hive下访问的格式为


	{
    "name": "John Doe",
    "salary": 100000.0 ,
    "subordinates": ["Mary Smith" , "Todd Jones"] ,   //列表Array, subordinates[1]=”Tood Jones”
    "deductions": {                                  //键值Map, deductions[’Federal Taxes’]=0.2
        "Federal Taxes": 0.2 ,
        "State Taxes": 0.05,
        "Insurance": 0.1
    }
    "address": {                                     //结构Struct, address.city=”Chicago”
        "street": "1 Michigan Ave." ,
        "city": "Chicago" ,
        "state": "IL" ,
        "zip": 60600
    }
	}

基于上述数据结构，我们在Hive里创建对应的表，并导入数据。

	John Doe,100000.0,Mary Smith_Todd Jones,Federal Taxes:0.2_State Taxes:0.05_Insurance:0.1,1 Michigan Ave._Chicago_1L_60600
	Tom Smith,90000.0,Jan_Hello Ketty,Federal Taxes:0.2_State Taxes:0.05_Insurance:0.1,Guang dong._China_0.5L_60661
	
	
Hive上创建测试表employees

	CREATE  TABLE learn.employees(
	name STRING,
	sa1ary FLOAT,
	subordinates ARRAY<STRING>,
	deductions MAP<STRING, FLOAT>,
	address STRUCT<street:STRING, city:STRING, 	state:STRING, zip:INT>
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','    -- 列分隔符
	COLLECTION ITEMS TERMINATED BY '_'  -- STRUCT 和 ARRAY 的分隔符
	MAP KEYS TERMINATED BY ':' -- MAP中的key与value的分隔符
	LINES TERMINATED BY '\n';  -- 行分隔符


导入文本数据到测试表


	load data local inpath "/home/hadoop/files/input/6_1.txt" overwrite into table learn.employees ;


访问三种集合列里的数据，以下分别是ARRAY，MAP，STRUCT的访问方式

	select subordinates[1], deductions['Federal Taxes'],address.city from learn.employees;


通过集合类型来定义列的好处是什么？

在大数据系统中，不遵循标准格式的一个好处就是可以提供更高吞吐量的数据。
当处理的数据的数量级是T 或者P 时，以最少的"头部寻址"来从磁盘上扫描数据是非常必要的。按数据集进行封装的话可以通过减少寻址次数来提供查询的速度。而如果根据外键关系关联的话则需要进行磁盘间的寻址操作，这样会有非常高的性能消耗。