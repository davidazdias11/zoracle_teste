# Oracle Database plugin
Provides native Zabbix solution for monitoring Oracle Database (multi-model database management system). 
It can monitor several Oracle instances simultaneously, remote or local to the Zabbix Agent.
The plugin keeps connections in the open state to reduce network congestion, latency, CPU and 
memory usage. Best for use in conjunction with the official 
[Oracle template.](https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/db/oracle_agent2) 
You can extend it or create your template for your specific needs. 

## Requirements
* Zabbix Agent 2
* Go >= 1.13 (required only to build from source)
* Oracle Instant Client >= 12

## Supported versions
* Oracle Database 12c2
* Oracle Database 18c
* Oracle Database 19c
* Possible all others :)

## Installation
* [Install Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client/downloads.html)
* Create an Oracle DB user and grant permissions 
```
CREATE USER zabbix_mon IDENTIFIED BY <PASSWORD>;
-- Grant access to the zabbix_mon user.
GRANT CONNECT, CREATE SESSION TO zabbix_mon;
GRANT SELECT ON DBA_TABLESPACE_USAGE_METRICS TO zabbix_mon;
GRANT SELECT ON DBA_TABLESPACES TO zabbix_mon;
GRANT SELECT ON DBA_USERS TO zabbix_mon;
GRANT SELECT ON SYS.DBA_DATA_FILES TO zabbix_mon;
GRANT SELECT ON V$ACTIVE_SESSION_HISTORY TO zabbix_mon;
GRANT SELECT ON V$ARCHIVE_DEST TO zabbix_mon;
GRANT SELECT ON V$ASM_DISKGROUP TO zabbix_mon;
GRANT SELECT ON V$DATABASE TO zabbix_mon;
GRANT SELECT ON V$DATAFILE TO zabbix_mon;
GRANT SELECT ON V$INSTANCE TO zabbix_mon;
GRANT SELECT ON V$LOG TO zabbix_mon;
GRANT SELECT ON V$OSSTAT TO zabbix_mon;
GRANT SELECT ON V$PGASTAT TO zabbix_mon;
GRANT SELECT ON V$PROCESS TO zabbix_mon;
GRANT SELECT ON V$RECOVERY_FILE_DEST TO zabbix_mon;
GRANT SELECT ON V$RESTORE_POINT TO zabbix_mon;
GRANT SELECT ON V$SESSION TO zabbix_mon;
GRANT SELECT ON V$SGASTAT TO zabbix_mon;
GRANT SELECT ON V$SYSMETRIC TO zabbix_mon;
GRANT SELECT ON V$SYSTEM_PARAMETER TO zabbix_mon;
```
* Make sure a TNS Listener and an Oracle instance are available for connection.  

## Configuration
The Zabbix agent 2 configuration file is used to configure plugins.

**Plugins.Oracle.CallTimeout** — The maximum time in seconds for waiting when a request has to be done.  
*Default value:* equals the global Timeout configuration parameter.  
*Limits:* 1-30

**Plugins.Oracle.ConnectTimeout** — The maximum time in seconds for waiting when a connection has to be established.  
*Default value:* equals the global Timeout configuration parameter.  
*Limits:* 1-30

**Plugins.Oracle.CustomQueriesPath** — Full pathname of a directory containing *.sql* files with custom queries.  
*Default value:* — (the feature is disabled by default)

**Plugins.Oracle.KeepAlive** — Sets a time for waiting before unused connections will be closed.  
*Default value:* 300 sec.  
*Limits:* 60-900

### Configuring connection
A connection can be configured using either keys' parameters or named sessions.     

*Notes*:  
* It is not possible to mix configuration using named sessions and keys' parameters simultaneously.
* You can leave any connection parameter empty, a default hard-coded value will be used in the such case.
* Embedded URI credentials (userinfo) are forbidden and will be ignored. So, you can't pass the credentials by this:   
  
      oracle.ping[tcp://USER:password@127.0.0.1/XE] — WRONG  
  
  The correct way is:
    
      oracle.ping[tcp://127.0.0.1,USER,password,XE]
      
* The only supported network schema for a URI is "tcp".  
Examples of valid URIs:
    - tcp://127.0.0.1:1521
    - tcp://localhost
    - localhost
* Only uppercase usernames are supported.
      
#### Using keys' parameters
The common parameters for all keys are: [ConnString][,User][,Password][,Service] 
Where ConnString can be either a URI or a session name.   
ConnString will be treated as a URI if no session with the given name is found.  
If you use ConnString as a session name, just skip the rest of the connection parameters.  
 
#### Using named sessions
Named sessions allow you to define specific parameters for each Oracle instance. Currently, there are only four
supported parameters: Uri, User, Password and Service.
It's a bit more secure way to store credentials compared to item keys or macros.  

E.g: suppose you have two Oracle instances: "Oracle12" and "Oracle19". 
You should add the following options to the agent configuration file:   

    Plugins.Oracle.Sessions.Oracle12.Uri=tcp://192.168.1.1:1521
    Plugins.Oracle.Sessions.Oracle12.User=<USERFORORACLE12>
    Plugins.Oracle.Sessions.Oracle12.Password=<PasswordForOracle12>
    Plugins.Oracle.Sessions.Oracle12.Service=orcl
        
    Plugins.Oracle.Sessions.Oracle19.Uri=tcp://192.168.1.2:1521
    Plugins.Oracle.Sessions.Oracle19.User=<USERFORORACLE19>
    Plugins.Oracle.Sessions.Oracle19.Password=<PasswordForOracle19>
    Plugins.Oracle.Sessions.Oracle19.Service=orcl
        
Then you will be able to use these names as the 1st parameter (ConnString) in keys instead of URIs, e.g:

    oracle.ping[Oracle12]
    oracle.ping[Oracle19]

*Note*: sessions names are case-sensitive.

## Supported keys
**oracle.custom.query[\<commonParams\>,query[,args...]]** — Returns result of a custom query.  
*Parameters:*  
query (required) — sql query to execute.  
args (optional) — one or more arguments to pass to a query.

So, you can execute them:
  
    oracle.custom.query[<commonParams>,'select 0 from dual']  
    oracle.custom.query[<commonParams>,'SELECT amount FROM payment WHERE user = :1 AND service_id = :2 AND date = :3',"John Doe",1,"10/25/2020"]
          
You can pass as many parameters to a query as you need.   
The syntax for placeholder parameters uses ":#", where "#" is an index number of a parameter.   

**oracle.ping[\<commonParams\>]** — Tests if connection is alive or not.  
*Returns:*
- "1" if a connection is alive.
- otherwise return the error as string. Ex: ORA-12545: Connect failed because target host or object does not exist.

     

## Current limitations
* Connection by SID is not supported.
* Only uppercase usernames are supported.

## Troubleshooting
The plugin uses Zabbix agent's logs. You can increase debugging level of Zabbix Agent if you need more details about 
what is happening.  
The environment variable DPI_DEBUG_LEVEL can be used to selectively turn on the printing of various logging messages
from ODPI-C. See [ODPI-C Debugging](https://oracle.github.io/odpi/doc/user_guide/debugging.html) for details.
