---
title: Linux下supervisor安装与使用
date: 2020-01-02 01:27:33
tags:
	- linux
	- supervision
categories: linux
---

# 一、centos7下安装supervisor
## **1.安装方法很多，举例一种**
```
yum install python-setuptools
easy_install supervisor
```
## **2.测试是否安装成功**
`echo_supervisord_conf`
## **3.新建supervisor文件夹**
`mkdir /etc/supervisor `
<!-- more -->
## **4.生成默认的配置文件**
`echo_supervisord_conf > /etc/supervisor/supervisord.conf`
## **5.然后编辑这个配置文件,配置成读取conf.d文件夹的配置文件,这样就不用写在一个文件里面：**
![img1](1.jpg)
## **6.同时将[inet_http_server]下的注释去掉,修改为：**
![img1](2.jpg)
## **7.配置文件：**
```
[root@centos-011 ~ 07:50:00]#cat /etc/supervisord.conf.bak
; Sample supervisor config file.
 
[unix_http_server]
file=/var/run/supervisor/supervisor.sock   ; socket 路径
 
;chmod=0700                 ; socket 文件的权限
;chown=nobody:nogroup       ; socket 所属用户及组
;username=user              ; 用户名
;password=123               ; 密码
 
[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
port=0.0.0.0:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
username=user              ; 登录管理后台的用户名
password=123               ; 登录管理后台的密码
 
[supervisord]               ; supervisord 全局配置
logfile=/var/log/supervisor/supervisord.log  ; supervisor 日志路径
logfile_maxbytes=50MB       ; 单个日志文件最大数
logfile_backups=10          ; 保留多少个日志文件（默认10个）
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; pid 文件路径
nodaemon=false              ; 启动是否丢到前台，设置为false ，表示以daemon 的方式启动
minfds=1024                 ; 最小文件打开数，对应系统limit.conf 中的nofile ,默认最小为1024，最大为4096
minprocs=200                ; 最小的进程打开数，对应系统的limit.conf 中的nproc,默认为200
;umask=022                  ; (process file creation umask;default 022)
;user=chrism                 ; 启动supervisord 服务的用户，默认为root
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; 这里的目录指的是服务的工作目录
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY=value       ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)
 
; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
 
[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket    ;通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available
 
; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.
 
;[program:theprogramname]      ; 定义一个守护进程 ，比如下面的elasticsearch 
;command=/bin/cat              ; 启动程序使用的命令，可以是绝对路径或者相对路径
;process_name=%(program_name)s ; 一个python字符串表达式，用来表示supervisor进程启动的这个的名称，默认值是%(program_name)s
;numprocs=1                    ; Supervisor启动这个程序的多个实例，如果numprocs>1，则process_name的表达式必须包含%(process_num)s，默认是1
;directory=/tmp                ; supervisord在生成子进程的时候会切换到该目录
;umask=022                     ; umask for process (default None)
;priority=999                  ; 权重，可以控制程序启动和关闭时的顺序，权重越低：越早启动，越晚关闭。默认值是999
;autostart=true                ; 如果设置为true，当supervisord启动的时候，进程会自动启动
;autorestart=true              ; 设置为随 supervisord 重启而重启，值可以是false、true、unexpected。false：进程不会自动重启
;startsecs=10                  ; 程序启动后等待多长时间后才认为程序启动成功，默认是10秒
;startretries=3                ; supervisord尝试启动一个程序时尝试的次数。默认是3
;exitcodes=0,2                 ; 一个预期的退出返回码，默认是0,2。
;stopsignal=QUIT               ; 当收到stop请求的时候，发送信号给程序，默认是TERM信号，也可以是 HUP, INT, QUIT, KILL, USR1, or USR2
;stopwaitsecs=10               ; 在操作系统给supervisord发送SIGCHILD信号时等待的时间
;user=chrism                   ; 如果supervisord以root运行，则会使用这个设置用户启动子程序
;redirect_stderr=true          ; 如果设置为true，进程则会把标准错误输出到supervisord后台的标准输出文件描述符
;stdout_logfile=/a/path        ; 把进程的标准输出写入文件中，如果stdout_logfile没有设置或者设置为AUTO，则supervisor会自动选择一个文件位置
;stdout_logfile_maxbytes=1MB   ; 标准输出log文件达到多少后自动进行轮转，单位是KB、MB、GB。如果设置为0则表示不限制日志文件大小
;stdout_logfile_backups=10     ; 标准输出日志轮转备份的数量，默认是10，如果设置为0，则不备份
;stdout_capture_maxbytes=1MB   ; 当进程处于stderr capture mode模式的时候，写入FIFO队列的最大bytes值，单位可以是KB、MB、GB
;stdout_events_enabled=false   ; 如果设置为true，当进程在写它的stderr
;stderr_logfile=/a/path        ; 把进程的错误日志输出一个文件中，除非redirect_stderr参数被设置为true
;stderr_logfile_maxbytes=1MB   ; 错误log文件达到多少后自动进行轮转，单位是KB、MB、GB。如果设置为0则表示不限制日志文件大小
;stderr_logfile_backups=10     ; 错误日志轮转备份的数量，默认是10，如果设置为0，则不备份
;stderr_capture_maxbytes=1MB   ; 当进程处于stderr capture mode模式的时候，写入FIFO队列的最大bytes值，单位可以是KB、MB、GB
;stderr_events_enabled=false   ; 如果设置为true，当进程在写它的stderr到文件描述符的时候，PROCESS_LOG_STDERR事件会被触发
;environment=A=1,B=2           ; 一个k/v对的list列表
;serverurl=AUTO                ; 是否允许子进程和内部的HTTP服务通讯，如果设置为AUTO，supervisor会自动的构造一个url
 
; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.
 # 这个地方是自定义一个守护进程
[program:elasticsearch]                       ; 定义一个守护进程 elasticsearch
environment=ES_HOME=/usr/local/elasticsearch  ; 设置ES_HOME 环境变量
user=elk                                      ; 启动elasticsearch 的用户
directory=/usr/local/elasticsearch            ; 进入到这个目录中
command=/usr/local/elasticsearch/bin/elasticsearch ; 执行启动命令
numprocs=1                                    ; Supervisor启动这个程序的多个实例，如果numprocs>1，则process_name的表达式必须包含%(process_num)s，默认是1
autostart=true                                ; 设置为随 supervisord 启动而启动
autorestart=true                              ; 设置为随 supervisord 重启而重启
startretries=3                                ; 设置elasticsearch 重启的重试次数
priority=1                                    ; 权重，可以控制程序启动和关闭时的顺序，权重越低：越早启动，越晚关闭。默认值是999  
 
;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; restart at unexpected quit (default: unexpected)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max #  of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max #  logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; #  of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max #  logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups        ; #  of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)
 
; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.
 
;[group:thegroupname]          ; 服务组管理，可以将多个服务名写到这里管理(组名自定义）
;programs=progname1,progname2  ; 上面配置好的服务名，比如elasticsearch,kibana,logstash
;priority=999                  ; the relative start priority (default 999)
 
; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.
 
[include]
files = /etc/supervisor/conf.d/*.conf
```
## **8.接下来就是编写执行命令了，在/etc/supervisor下新建conf.d文件夹，在里面新建一个conf文件，命令内容如下(`注意前后不能有空格`)。一般放在：/etc/supervisor/conf.d/目录 。一个脚本对应一个配置文件。**
```
[program:yourprogramname]
command=python meipaiUser.py
directory=/home/tomcat/pyscripts/recommendation/videoWebsiteCrawler/videoWebsiteCrawler  ;项目启动目录
stdout_logfile=/home/tomcat/pyscripts/recommendation/videoWebsiteCrawler/meipaiuser.log    ;日志输出目录
;autostart = true     ; 在 supervisord 启动的时候也自动启动
;startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
;autorestart = true   ; 程序异常退出后自动重启
;startretries = 3     ; 启动失败自动重试次数，默认是 3
;user = leon          ; 用哪个用户启动
;redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
;stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
;stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```
## **9.启动Supervisor服务**
`supervisord -c /etc/supervisor/supervisord.conf   或  supervisord`
## **10.任务管理**
### 1.`supervisorctl`命令进入 supervisorctl 的 shell 界面，然后可以执行不同的命令了：
```
> status    ＃查看正在运行的程序状态
> stop program_name   ＃关闭 program_name 程序
> start program_name   ＃启动 program_name 程序
> restart program_name     ＃重启 program_name 程序
> reload     ＃读取有更新（增加）的配置文件，不会启动新添加的程序
> update    ＃重启配置文件修改过的程序
> stop all    ＃关闭所有程序

-----------------------------------------

supervisorctl stop all，停止全部进程，注：start、restart、stop都不会载入最新的配置文件。
supervisorctl reload，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
supervisorctl update，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启
```
注：显示用stop停止掉的进程，用reload或者update都不会自动重启。
### 2.也可以通过web界面，出于安全考虑，默认配置是没有开启web管理界面，需要修改supervisord.conf配置文件打开http访权限：
```
[inet_http_server]         ; inet (TCP) server disabled by default
port=0.0.0.0:9001          ; (ip_address:port specifier, *:port for all iface)
username=user              ; (default is no username (open server))
password=123               ; (default is no password (open server))
```
访问`http://ip:9001`即可进行任务管理。**（如果访问不了，记得把Linux的防火墙关闭）**
查看自己的程序是否运行成功
`ps -ef | grep supervisor`  

# 二、开机启动Supervisor服务  
## **1.配置systemctl服务**
**进入/lib/systemd/system目录，并创建supervisord.service文件**
```
[Unit]
Description=supervisor
After=network.target

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
## **2.设置开机启动**
```
systemctl daemon-reload
systemctl enable supervisord.service
```
## **3.修改文件权限为766**
`chmod 766 supervisord.service`
## **4.配置service类型服务**
```
#!/bin/bash
# 
#  supervisord   This scripts turns supervisord on
# 
#  Author:       Mike McGrath <mmcgrath@redhat.com> (based off yumupdatesd)
# 
#  chkconfig:    - 95 04
# 
#  description:  supervisor is a process control utility.  It has a web based
#                xmlrpc interface as well as a few other nifty features.
#  processname:  supervisord
#  config: /etc/supervisor/supervisord.conf
#  pidfile: /var/run/supervisord.pid
# 

#  source function library
. /etc/rc.d/init.d/functions

RETVAL=0

start() {
    echo -n $"Starting supervisord: "
    daemon "supervisord -c /etc/supervisor/supervisord.conf "
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/supervisord
}

stop() {
    echo -n $"Stopping supervisord: "
    killproc supervisord
    echo
    [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/supervisord
}

restart() {
    stop
    start
}

case "$1" in
  start)
    start
    ;;
  stop) 
    stop
    ;;
  restart|force-reload|reload)
    restart
    ;;
  condrestart)
    [ -f /var/lock/subsys/supervisord ] && restart
    ;;
  status)
    status supervisord
    RETVAL=$?
    ;;
  *)
    echo $"Usage: $0 {start|stop|status|restart|reload|force-reload|condrestart}"
    exit 1
esac

exit $RETVAL
```
## **5.将上述脚本内容保存到/etc/rc.d/init.d/supervisor文件中，修改文件权限为755，并设置开机启动**
```
chmod 755 /etc/rc.d/init.d/supervisord
chkconfig supervisor on
```
**注意：**
**`Supervisor只能管理非daemon的进程，也就是说Supervisor不能管理守护进程。否则提示Exited too quickly(process log may have details)异常。例子中的Tomcat默认是以守护进程启动的，所以我们改成了catalina.sh run，以前台进程的方式运行。`**

**参考：https://www.cnblogs.com/heyongboke/p/9188125.html**