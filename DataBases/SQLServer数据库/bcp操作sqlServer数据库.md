* [sqlserver中的bcp查询导出海量数据](https://blog.csdn.net/zhangshufei8001/article/details/51130072)
 * 在SQLServer中要开启组件 'xp_cmdshell',要不就报错,因为为了安全,sql默认是关闭的.
错误信息:在执行上述命令的时候可能会报错 ： 错误提示：消息 15281，级别 16，状态 1，过程 xp_cmdshell，第 1 行
打开SQLServer数据库，新建查询语句，输入以下命令：
```sql
-- 允许配置高级选项  
EXEC master.sys.sp_configure 'show advanced options', 1  
-- 重新配置  
RECONFIGURE  
-- 启用xp_cmdshell  
EXEC master.sys.sp_configure 'xp_cmdshell', 1  
--重新配置  
RECONFIGURE
```
* [bcp 实用工具-msdn](https://docs.microsoft.com/zh-cn/sql/tools/bcp-utility?view=sql-server-2014)
* [指定字段终止符和行终止符 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/relational-databases/import-export/specify-field-and-row-terminators-sql-server?view=sql-server-2014)
* [创建格式化文件 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/relational-databases/import-export/create-a-format-file-sql-server?view=sql-server-2014)
* [用来导入或导出数据的格式化文件 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/relational-databases/import-export/format-files-for-importing-or-exporting-data-sql-server?view=sql-server-2014)
* [XML 格式化文件 (SQL Server)](https://docs.microsoft.com/zh-cn/sql/relational-databases/import-export/xml-format-files-sql-server?view=sql-server-2014)
* [Transact-SQL 语法约定 (Transact-SQL)](https://docs.microsoft.com/zh-cn/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql?view=sql-server-2017)
* [SQL Server中BCP导入导出用法详解](http://blog.sina.com.cn/s/blog_5ceb51480101gtr5.html)
* [SQL中通过UTS和BCP批量导入数据到数据库详解](http://blog.sina.com.cn/s/blog_5ceb51480101gyqd.html)
* [BCP命令提示实用工具](https://blog.csdn.net/u012968272/article/details/45418867)
* [BCP工具的使用以及C++，SQL server数据库中调用命令行的方法](https://blog.csdn.net/a342500329a/article/details/83896518)
* [SQL Server中bcp命令的用法以及数据批量导入导出](https://www.cnblogs.com/xwdreamer/archive/2012/08/22/2651180.html)]()
* [SQL Server大表数据的导出与导入命令BCP](https://blog.csdn.net/iteye_11587/article/details/82681646)
* [BCP命令，导入导出CSV文件](https://blog.csdn.net/weixin_42126947/article/details/80513220)
* [SQL Server BCP使用小结](http://www.cnblogs.com/qanholas/archive/2011/07/05/2098616.html)


