234390216 的留下学习 原文地址 http://haohaoxuexi.iteye.com/blog/1594391


 Oracle的闪回功能可以在对数据库进行不完全恢复的情况下，对某一个指定的表进行恢复。闪回数据库是进行时间点恢复的新方法，它能够快速将Oracle恢复到以前的时间，以更正由于逻辑数据损坏或用户错误而引起的问题。当需要恢复时，可以将数据库恢复到错误前的时间点，并且只恢复改变的数据块。<br/>
 Oracle中的闪回操作包括以下4种：<br/>
 （1）查询闪回：查询过去某个指定时间、指定实体的数据，恢复错误的数据库更新、删除等。<br/>
 （2）表闪回：使表返回到过去的某一时间的状态，恢复表、取消对表进行的修改。<br/>
 （3）删除闪回：可以将删除的表重新恢复。<br/>
 （4）数据库闪回：可以将整个数据库回退到过去的某个时间点。<br/>
 
 1、查询闪回<br/>
 查询闪回可以查看过去某一时点的任何数据，如果要查询某一表在某一时点的内容，可以把查询目标对象定位为该表在某一时点的表，表在某一时刻的表可以如下表示：

Sql代码  
```
table_name as of timestamp real_timestamp; --它作为一个整体表示一个表*/  
```
例如，要查询person表在2012-6-2 19:00:00的状态，可以使用如下语句：
```
select * from person as of timestamp to_timestamp('2012-6-2 19:00:00', 'yyyy-mm-dd HH24:mi:ss');  
```
知道了表在某一时刻的表之后，我们就可以很容易的把表恢复到某一时刻的样子了，例如我们在2012-6-2 19:00:00这个时候删除了person表中的3条记录，之后我又想把它恢复，那应该如何恢复呢？利用查询闪回我们可以这样操作：<br/>
##### 方法一：<br/>
###### 第一步，把当前表中的数据全删了：delete from person;<br/>
###### 第二步，我们通过查询闪回把2012-6-2 19:00:00这个时点之前的数据拿出来然后插入到当前表中，调用语句如下：<br/>
```
insert into person select * from person as of timestamp to_timestamp('2012-6-2 18:59:59', 'yyyy-mm-dd HH24:mi:ss');  
```
##### 方法二：
先找出从person表中删除的记录，然后再把它们插到person表中，Sql语句如下：
```
insert into person select * from person as of timestamp to_timestamp('2012-6-2 18:59:59', 'yyyy-mm-dd HH24:mi:ss') p where not exists(select * from person where id=p.id);  
```
 
因为查询闪回是跟时间有关系的，所以说到这就再说一个设置，在sqlplus中输入set time on语句可以打开时间显示功能，这会使得在每次执行语句后都会在该语句的前面显示该语句的执行时间，使用set time off语句可以关闭该功能。
 
 2、表闪回
利用表闪回可以轻松的取消对表的修改。只有Oracle的企业版才能进行表闪回。
对进行表闪回的表必须row movement为enable。<br/>
表闪回的语法格式如下：<br/>
```
flashback table [schema.]table_name[,...n] to {[scn] | [timestamp] [[enable | disable] triggers]};  
```
其中，
`scn`：表示系统改变号，可以从flashback_transaction_query数据字典中查询。<br/>
`timestamp`：表示通过时间戳的形式来进行闪回。<br/>
`enable|disable triggers`：表示触发器恢复之后的状态，默认为disable。<br/>
示例代码：<br/>
######（1）确保需要闪回的表row movement为enable：
```
alter table hello enable row movement;  
```
###### （2）对表进行闪回：
```
flashback table hello to timestamp to_timestamp('2012-6-3 14:00:00','yyyy-mm-dd HH24:mi:ss');--恢复表到2012-6-3 14:00:00这个时候的样子*/  
```
 
###### (3) 删除闪回
在Oracle数据库中有一个叫recyclebin的回收站，当回收站的功能是启用的时候，被用户drop的对象都会保存在recyclebin中，普通用户可以通过select * from recyclebin；语句查看被自己drop掉的对象的相关信息，当然如果普通用户想查看所有用户的recyclebin信息也是可以的，可以通过查看dba_recyclebin视图，DBA用户不存在recyclebin。来获取，如：select * from dba_recyclebin。如果被删除的对象不是彻底删除，而是放到了回收站的话，我们就可以在需要的时候从回收站中进行恢复了。<br/>
为了避免被删除的表与其他对象存在名称冲突，Oracle对被删除的对象进行了重命名，当然Oracle也保存了对象的原始名称。在进行对象恢复的时候可以使用对象的原始名称，也可以使用被重新命名的新名称。<br/>
系统参数recyclebin控制表删除后是否到回收站，show parameter recyclebin可以查看该参数的状态。<br/>
对于系统参数的修改有两种，全局的修改和会话的修改：<br/>
```
（1）alter system set param_name=param_value;
（2）alter session set param_name=param_value;
 show recyclebin：可以显示当前用户recyclebin中的表。
 purge tablespace tablespace_name：用于清空指定表空间的recyclebin；
 purge tablespace tablespace_name user username：清空指定表空间的recyclebin中指定用户的对象。
 purge table table_name：清除回收站中的指定表对象。如：purge table hello语句则将清除回收站中的hello表。这里的table_name既可以是原来的表对象名，也可以是
 recyclebin中自动生成的名字。
 purge recyclebin：删除当前用户的recyclebin中的对象。
 purge dba_recyclebin：删除所有用户的recyclebin中的对象
 drop table table_name purge：删除对象且不放在recyclebin中。
```
 删除闪回的语法格式如下：
```
flashback table table_name to before drop [rename to new_name]; --恢复表table_name并重命名为new_name*/  
```
示例代码：
```
flashback table hello to before drop rename to dropped_hello;  
```
###### (4) 数据库闪回
 数据库闪回可以使数据库回到过去某一时间点或SCN的状态，用户可以不用备份就能快速地实现时间点的恢复。只有Oracle的企业版才能进行数据库闪回。