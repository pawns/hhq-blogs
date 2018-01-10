# hhq-blogs
## mysql 索引
##### 如何建索引
####  (一)  : 区分度
1.先计算该字段的区分度
区分度: 指字段在数据库中的不重复比 字段去重后的总数与全表总记录数的商 0.0000-1.0000之间，区分度越高索引效果越佳
select count(distinct(name))/count(*) from t_base_user;
#####  1.单列索引
可以查看该字段的区分度,根据区分度的大小,也能大概知道在该字段上的新建索引是否有效，以及效果如何。区分度越大,索引效果越明显。
##### 2.多列索引(联合索引)
多列索引中其实还有一个字段的先后顺序问题,一般是将区分度较高的放在前面,这样联合索引才更有效,例如:
select * from t_base_user where name="" and status=1;
像上述语句,如果建联合索引的话,就应该是:
alter table t_base_user add index idx_name_status(name,status);
而不是:
alter table t_base_user add index idx_status_name(status,name)；

假设存在组合索引it1c1c2(c1,c2)，查询语句select * from t1 where c1=1 and c2=2能够使用该索引。查询语句select * from t1 where c1=1也能够使用该索引。但是，查询语句select * from t1 where c2=2不能够使用该索引，因为没有组合索引的引导列，即，要想使用c2列进行查找，必需出现c1等于某值
#### (二) 最左前缀匹配原则
MySQL会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如
select * from t_base_user where type="10" and created_at<"2017-11-03" and status=1, (该语句仅作为演示)
在上述语句中,status就不会走索引,因为遇到<时,MySQL已经停止匹配,此时走的索引为:(type,created_at),其先后顺序是可以调整的,而走不到status索引,此时需要修改语句为:
select * from t_base_user where type=10 and status=1 and created_at<"2017-11-03"
即可走status索引。
#### (三) 函数运算
  不要在索引列上,进行函数运算,否则索引会失效。因为b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。

#### (四) 扩展优先
  扩展优先,不要新建索引,尽量在已有索引中修改。如下:
select * from t_base_user where name="andyqian" and email="andytohome"
在表t_base_user表中已经存在idx_name索引,如果需要加入idx_name_email的索引,应该是修改idx_name索引,而不是新建一个索引。
