---
layout: post
tId: 1605161
title: "项目中常用的Sql查询-分页递归聚合和条件查询"
date: 2016-05-16 13:00:00 +0800
categories: 数据库
codelang: sql
desc: "大多数使用关系型数据库的项目都离不开sql查询，对于业务开发来说，sql查询封装完成之后，便少有接触了，特将数据库中较多使用的点进行汇总，以便于进行知识回顾和速查即用"
---
为了便于在以后的项目中借鉴引用，特将项目中使用的关键性sql查询进行下汇总

### 分页
sql server 2008之后支持通过FETCH NEXT实现分页功能，不再需要通过rownumber的方式实现了

通过使用存储过程实现分页查询,示例代码是对Article文章的分页查询实现，繁杂的代码主要集中在where后的条件构造，对于传入值为null的项，需要进行条件忽略

```
CREATE PROCEDURE [dbo].[proce_ArticlePaging]
-- Add the parameters for the stored procedure here
	@itemType smallint,
	@beginTime Nvarchar(30),
	@endTime Nvarchar(30),
	@peopleId int,
	@articleStatus smallint,
	@pageSize int,
	@pageIndex int,
	@totalCount int output
AS
BEGIN
	declare @whereCondition nvarchar(1000);
	Set @whereCondition=N' a.deleted=0 ';
	if(@articleStatus>=0)
	Begin
	  Set @whereCondition +=' and a.status='+LTRIM(@articleStatus);
	End
	if(@itemType>=0 or @beginTime!=N''or @endTime!=N'')
	 Begin
	   declare @needAnd int;
	   Set @needAnd=0;
	   Set @whereCondition +=N' and a.ProjectId in ( select ProjectId from Project where ';
	    if(@itemType>=0)
		 Begin
		  Set @whereCondition+= N' exists(select projectItemId from ProjectItem where itemType='+LTRIM(@itemType)+') ';
		  Set @needAnd=1;
		 End
		if(@beginTime!=N'')
		 Begin
		  if(@needAnd=1) Set @whereCondition+=N' and ';
		  Set @whereCondition+=N' createTime > '''+@beginTime+'''';
		  Set @needAnd=1;
		 End
		if(@endTime!=N'')
		Begin
		  if(@needAnd=1) Set @whereCondition+=N' and ';
		  Set @whereCondition+=N' createTime >'''+@endTime+'''';
		End
	   Set @whereCondition +=N' ) ';
	 End
	 --
	 declare @sqlCount nvarchar(1000);
	 declare @sqlQuery nvarchar(1000);
	 Set @sqlCount='select @totalCount=count(*) from Article a where '+@whereCondition;
	 Set @sqlQuery='select [articleId],[title],[deleted],a.[status],a.[createTime],[personId]
				  ,[publishTime]
				  ,[tags]
				  ,[lastModified]
				  ,[projectId]
				  ,[lastModifiedBy]
				  ,[projectItemId]
				  ,[applyId]
				  ,ap.userName
				  ,ap.memo from Article a left join ApprovalProgress ap on a.applyId=ap.itemId and ap.itemType=1 where '+@whereCondition
				  +' order by  a.createTime desc OFFSET '+LTRIM(@pageSize*(@pageIndex-1))+N' ROWS FETCh NEXT '+LTRIM(@pageSize)+N' ROWS ONLY';
      EXEC sp_executeSql @sqlCount,N'@totalCount int output',@totalCount output;
	  print @sqlQuery;
	  EXEC sp_executesql @sqlQuery;
END
```

### id字符集合转换
很多时候，我们在根据id进行筛选的时候，都会传入一个用','分割的字符串，如果需要筛选的id过多，导致最终拼接的sql语句过长，影响效率甚至导致查询失败，这个时候我们实现一个转换函数，就id的字符串转化为包含这些id的一张表

```
ALTER function [dbo].[f_split](
    @s varchar(max)
    ,@Quote varchar(5)
)
returns @t TABLE (item int primary key  clustered)
AS
BEGIN
declare @xml xml
select @xml='<r s="'+replace(@s,@Quote,'"></r><r s="')+'"></r>'
insert @t SELECT distinct X.n.value('@s','varchar(256)') FROM @xml.nodes('/r') AS X(n)
return;
END
```

通过上述方法，查询就变成了表与表之间的查询

```
select * from ArticleId IN (SELECT item FROM f_split(@articleIds, ','))
```

### 递归查询
当需要查询某个id的所有父级节点信息时，可以通过递归的方法，进行查询。

```
public static string GenerateParentFindSqlStr(string tableName,string idField,int id)
{
    return string.Format(@"with cte as
                    (
                    select * from {0} where {1}={2}
                    union all
                    select a.* from {0} a join cte b on a.{1}=b.ParentId
                    )select * from cte", tableName, idField,id);
}
```

### 聚合查询
我们大都这么用过以下方法：

```
select count(*) from table
```

来查询集合的数量，count属于聚合函数，其他的也有如AVG、MAX、MIN、SUM等等，这种情况很简单，然而当我们需要通过group by进行汇总查询时：

```
select Title from Article group by ArticleType
```

*选择列表中的列 'Article.Title' 无效，因为该列没有包含在聚合函数或 GROUP BY 子句中*

根据错误信息调整如下（示例无实际意义）：

```
select Title from Article group by Title
--或者
select count(Title) from Article group by ArticleType
```

也就是查询的字段要不包含在聚合函数中，要不包含在group by下，否则在group by的查询中，就会出错

### case when 条件查询
曾经涉及到一个查询条件是汇总一个项目中子项目的不同状态数量，为了方便查询，使用了case when的条件查询，并进行了汇总

```
//需要查询的状态
Model.ApprovalStatusEnum[] statusArr = new Model.ApprovalStatusEnum[] { ApprovalStatusEnum.Check};
StringBuilder sb = new StringBuilder();
sb.Append("select projectId");
for (int i = 0; i < statusArr.Length; i++)
{
    sb.Append(",SUM(case when status=");
    sb.Append(Convert.ToInt16(statusArr[i]));
    sb.Append(" then 1 Else 0 End) as Statistic");
    sb.Append((i + 1));
}
sb.Append(" from ProjectApply where projectid in (");
sb.Append(projectIds);//projectIds如："1,2,3,4,5"
sb.Append(") group by projectId");
```

实际statusArr是通过参数传递的，因此调用时可以任意组合想要查询的状态，返回结果为Statistic1,Statistic2....
返回结果需要根据传入的状态，进行一次解读，这也是提高复用性之后一个小小的不便。

### 总结

以上是在近期项目中常用的查询或相关知识点的汇总，以便于复用和加深印象。sql的一些复杂用法实际并不是时时在使用（DBA除外），实际开发中更多的是业务相关的开发，大部分的sql操作，前期都会封装好了，所以多多回顾是很有必要的。

