### 转自：http://blog.csdn.net/dba_huangzj/article/details/8426684
## 起因：
由于工作原因，我隔几天就要执行一批开发人员提供过来的脚本，部分是新需求的开发，部分是修复bug。往往包含有几百个。我用工具批量执行之后，系统继续运行，后来反反复复会有这样那样的错误，其中一个，经过开发人员的检查，是因为视图没刷新。
对此我纳闷了很久，视图不就是一堆select语句吗？怎么还要刷新？难道表改了不会跟着改？为此，我首先自己做一个实验，发现的确不会马上改过来，至于啥时候才更改，也不清楚，听说从2000的时候，这个问题已经存在，看来我孤陋寡闻了。

## 测试：

### 步骤一：首先执行下面语句

```
[sql] view plain copy print?
USE tempdb  
GO  
--创建表  
IF OBJECT_ID('testTB') IS NOT NULL   
    DROP TABLE testTB  
GO  
CREATE TABLE testTB ( id INT, NAME VARCHAR(10) )  
--插入测试数据  
INSERT INTO testTB  
SELECT 1,'a'  
UNION ALL   
SELECT 2,'b'  
UNION ALL   
SELECT 3,'c'  
  
IF OBJECT_ID('V_testTB') IS NOT NULL   
DROP VIEW V_testTB  
GO  
CREATE   VIEW V_testTB  
AS  
    SELECT  *  
    FROM    testTB  
go   
  
SELECT * FROM V_testTB  
```
得到结果：




### 步骤二：更改表结构
```
[sql] view plain copy print?
--添加一列  
ALTER TABLE testTB ADD  age INT  
```
然后再来执行一下视图：
```
[sql] view plain copy print?
SELECT * FROM V_testTB  
```
得到结果：



反复执行了10次，结果还是没变。

### 步骤三：使用存储过程刷新视图
```
[sql] view plain copy print?
sp_refreshview V_testTB  
```
然后再执行查询视图的语句：
```
[sql] view plain copy print?
SELECT * FROM V_testTB  
```
眼前一亮，得到结果：



可以看出，结构已经刷新，证明有效果了。

### 分析：

细心的人应该发现，其实视图里面我用了\*号。以通过实验来证明，如果不用星号，是没问题的。而如果指定了列名，那么在新加一列的时候，管它有没有刷新，都不会有问题，因为你压根就不会用到这列，那么如果是删除呢？现在来试试，建表的代码依旧，把原有的添加列的代码改成删除列，另外\*号依旧保留：
```
[sql] view plain copy print?
--删除一列  
ALTER TABLE testTB DROP   COLUMN  id  
```
再执行：
```
[sql] view plain copy print?
SELECT * FROM V_testTB  
```
会得到以下的错误：

![错误信息](http://img.my.csdn.net/uploads/201212/24/1356358342_9684.png)
为视图或函数'V_testTB'指定的列名比其定义中的列多

证明删除是会报错的，不需要刷新，那么估计大家也猜到，就算指定列，也会报错，现在来证实一下：

![错误信息](http://img.my.csdn.net/uploads/201212/24/1356358492_3983.png)
列名'id'无效
由于绑定错误，无法使用视图或函数'V_testTB'

这次报错是这个，部分代码我就不写了。

## 总结：

根据上面的实验，可以得出：
 1、视图里面尽可能不要出现*号。*号不仅对性能有影响，也不便于结构的更新。
       
 2、无论视图所涉及的表结构有无修改，每次执行脚本后，刷新一下，总是好的。并且我遇到过这样的情景，一个名字是存储过程的名字，但是在使用：

[sql] view plain copy print?
SELECT DISTINCT  
        'EXEC sp_refreshview ''' + name + ''''  
FROM    sys.objects AS so  
        INNER JOIN sys.sql_expression_dependencies AS sed ON so.object_id = sed.referencing_id  
WHERE   so.type = 'V'  
        AND sed.referenced_id = OBJECT_ID('testTB') ;  

下面语句中时竟然能查出来，证明定义的时候有问题，所以这一步也同时可以检查一下会不会存在问题对象。顺带说一句，上面的脚本是把需要刷新的视图拼接出来，然后一次性执行。
