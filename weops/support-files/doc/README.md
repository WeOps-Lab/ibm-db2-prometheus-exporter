## 嘉为蓝鲸DB2插件使用说明

## 使用说明

### 插件功能
连接数据库使用SQL采集数据转换为监控指标

### 版本支持

操作系统支持: linux
 
是否支持arm: 不支持  
DB2 driver不兼容ARM64架构  

**组件支持版本：**

DB2版本: 9.7+

**是否支持远程采集:**

是

**注意**  
1. 一般部署db2的机器操作系统版本比较低，可能会出现依赖不满足探针的情况，建议使用操作系统较新的服务器做远程采集。
2. 一个采集任务只能连接一个数据库，如果想要采集多个数据库，需要启动多个采集任务。
3. 如果有报错，可能是数据库限制的某些数据的访问，需要按照日志内容进行处理。

### 参数说明


| **参数名**           | **含义**                          | **是否必填** | **使用举例**       |
|-------------------|---------------------------------|----------|----------------|
| LD_LIBRARY_PATH   | DB2客户端驱动程序的lib目录(环境变量)，建议使用绝对路径 | 是        | /clidriver/lib |
| DATABASE_HOSTNAME | 数据库服务IP(环境变量)                   | 是        | 127.0.0.1      |
| DATABASE_PORT     | 数据库服务端口(环境变量)                   | 是        | 50000          |
| DATABASE_USER     | 数据库用户名(环境变量)                    | 是        | weops          |
| DATABASE_PASSWORD | 数据库密码(环境变量)                     | 是        | Weops123!      |
| DATABASE_NAME     | 数据库名称(环境变量)                     | 是        | weopsdb        |
| --log.level       | 日志级别                            | 否        | error          |


### 使用指引
#### 前置依赖  
LD_LIBRARY_PATH的路径是DB2连接驱动，需要解压 `linuxx64_odbc_cli.tar.gz` 到指定目录，解压后会出现 `clidriver` 目录，填写好路径(clidriver下的lib)到变量中才可运行该插件。    

#### 配置权限  
1. 创建操作系统组  
- Linux系统: `sudo groupadd db2mons`  
- Windows系统:  
    打开 "Computer Management" -> "Local Users and Groups" -> "Groups"  
    右键点击 "Groups" 然后选择 "New Group" 来创建 db2mons 组  

2. 新建用户并将用户添加到该组 
- Linux系统: 
需要在root下执行  
```shell
    useradd db2monitor
    passwd db2monitor  
    usermod -a -G db2mons db2monitor 
``` 

- Windows系统:
    打开 "Computer Management" -> "Local Users and Groups" -> "Groups"  
    找到 db2mons 组并将 db2monitor 用户添加到该组中  


3. 更新DB2配置   
使用超管用户下的连接，配置系统监控组为 `db2mons`  
```shell
# 连接到超管用户
su - {超管用户}
db2 connect to {数据库名称} user {超管用户} using {超管密码}
db2 update dbm cfg using SYSMON_GROUP db2mons
```


db2日志显示系统监控组配置为 `db2mons`  
```
2024-10-21-03.54.39.596833+000 I270000E319           LEVEL: Event
PID     : 18868                TID : 140549449732032 PROC : db2flacc
INSTANCE: db2inst1             NODE : 000
HOSTNAME: db2-0
FUNCTION: DB2 UDB, config/install, sqlfLogUpdateCfgParam, probe:30
CHANGE  : CFG DBM: "Sysmon_group" From: ""  To: "db2mons" 
```

4. 重新启动DB2实例使更改生效
```shell
db2stop
db2start
```

5. 确认配置是否生效
下方是正常配置后的返回信息，可以看到DB2MONS是监控权限组
```shell
[db2inst1@db2-0 ~]$ db2 get dbm cfg | grep SYSMON_GROUP
 SYSMON group name                        (SYSMON_GROUP) = DB2MONS 
```


连接的数据库必须激活
```shell
db2
activate database {数据库名称}
quit
```

6. 检查授权详情  
示例用户 `DB2MONITOR` 和 `DB2MONITOR1`都可正常抓取监控指标  
```shell
db2 "select * from syscat.dbauth"

GRANTOR                                                                                                                          GRANTORTYPE GRANTEE                                                                                                                          GRANTEETYPE BINDADDAUTH CONNECTAUTH CREATETABAUTH DBADMAUTH EXTERNALROUTINEAUTH IMPLSCHEMAAUTH LOADAUTH NOFENCEAUTH QUIESCECONNECTAUTH LIBRARYADMAUTH SECURITYADMAUTH SQLADMAUTH WLMADMAUTH EXPLAINAUTH DATAACCESSAUTH ACCESSCTRLAUTH CREATESECUREAUTH
-------------------------------------------------------------------------------------------------------------------------------- ----------- -------------------------------------------------------------------------------------------------------------------------------- ----------- ----------- ----------- ------------- --------- ------------------- -------------- -------- ----------- ------------------ -------------- --------------- ---------- ---------- ----------- -------------- -------------- ----------------
SYSIBM                                                                                                                           S           DB2INST1                                                                                                                         U           N           N           N             Y         N                   N              N        N           N                  N              Y               N          N          N           Y              Y              N               
SYSIBM                                                                                                                           S           PUBLIC                                                                                                                           G           Y           Y           Y             N         N                   Y              N        N           N                  N              N               N          N          N           N              N              N               
DB2INST1                                                                                                                         U           DB2MONITOR                                                                                                                       U           N           Y           N             N         N                   N              N        N           N                  N              N               N          N          N           Y              N              N               
DB2INST1                                                                                                                         U           DB2MONITOR1                                                                                                                      U           N           Y           N             N         N                   N              N        N           N                  N              N               N          N          N           Y              N              N               

  4 record(s) selected.
```

### 指标简介


### 版本日志

#### weops_DB2_exporter 1.2.2

- weops调整
- 新增指标 ibm_db2_tablespace_used_percent 已使用表空间百分比
