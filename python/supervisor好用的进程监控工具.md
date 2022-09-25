# supervisor好用的进程监控工具

### 什么是 supervisor

supervisor 是一个用 Python 语言编写的进程管理工具，它可以很方便的监听、启动、停止、重启一个或多个进程。当一个进程意外被杀死，supervisor 监听到进程死后，可以很方便地让进程自动恢复，不再需要程序猿或系统管理员自己编写代码来控制。





### 安装 supervisor

安装方式很简单，直接 pip3 install supervisor 即可，但是注意：supervisor 不支持 Windows。

安装完毕之后会生成 3 个系统命令：supervisord、supervisorctl、echo_supervisord_conf，作用如下：

- `supervisord：负责启动进程并将其作为自己的子进程来管理，当管理的进程出现崩溃时能够自动重启`
- `supervisorctl：命令行管理工具，可以通过 start、stop、restart 命令来操作相应的子进程，比如启动、停止、重启等等`
- `echo_supervisord_conf：我们在启动 supervisor 的时候可以指定配置文件，而 echo_supervisord_conf 可以生成默认的配置文件(里面内容非常齐全并且有注释，所以更常被用作查阅)`

下面来介绍一下这几个命令的使用方式。



#### supervisord

我们说它是负责管理进程的，但是它本身也是一个进程，所以我们需要先启动它。启动之后，supervisorctl 所做的一系列操作都会交给 supervisord。

```
supervisord -c 配置文件
```

当我们写好一个配置文件之后，通过 supervisord -c 即可启动了，比较简单。但是除了 -c 之外，我们还会搭配其它的参数使用，比如：

- `-n：以前台方式运行(supervisord -n -c 配置文件)`
- `-s：日志不输出到控制台(supervisord -s -c 配置文件)`
- `-u user：以指定用户运行(supervisord -s -u root 配置文件)`

具体都有哪些参数，可以通过 supervisord --help 进行查看，但是里面的大部分参数都可以在配置文件中指定。



#### supervisorctl

supervisorctl 是使用率最频繁的命令了，我们在启动 supervisord 之后，实际上都会使用 supervisorctl 来操作进程。支持的操作如下：

- `supervisorctl start program_name：启动某个进程`
- `supervisorctl stop program_name：停止某个进程`
- `supervisorctl restart program_name：重启某个进程`
- `supervisorctl status program_name：查看某个进程的状态`
- `supervisorctl stop all：停止全部进程`
- `supervisorctl reload：载入最新的配置文件，重启所有进程`
- `supervisorctl update：根据最新的配置，重启配置更改过的进程，未更新的进程不受影响`



#### echo_supervisord_conf

它是用来生成默认的配置文件的，我们在命令行输入 echo_supervisord_conf 即可打印相关内容，所以可以通过 echo_supervisord_conf > supervisord.conf 写入到配置文件中，因此我们重点是来看看这个配置文件里面都有哪些内容，以及含义是什么？

里面可配置的选项还是非常多的，但是实际不一定每一个都要用到。

```
; 里面的的 ; 表示注释, 我们需要使用哪一个选项就直接把注释打开即可

[unix_http_server]
file=/tmp/supervisor.sock   ; socket 文件路径
;chmod=0700                 ; socket 文件的权限
;chown=nobody:nogroup       ; socket 文件所属的用户和组
;username=user              ; 用户名
;password=123               ; 密码


;[inet_http_server]         ; 是否启动 webUI 服务，启动的话可以通过浏览器查看 supervisor 管理的服务状态，默认是被注释掉的
;port=127.0.0.1:9001        ; 监听的 ip 以及端口
;username=user              ; 用户名
;password=123               ; 密码

[supervisord]                ; supervisord 的全局配置
logfile=/tmp/supervisord.log ; supervisord 的日志路径
logfile_maxbytes=50MB        ; 单个日志文件的最大体积
logfile_backups=10           ; 保留多少个日志文件，默认 10 个
loglevel=info                ; 日志等级，其它还有 debug,warn,trace
pidfile=/tmp/supervisord.pid ; supervisord 的 pid 文件路径
nodaemon=false               ; 是否前台启动，显然这个参数就是启动 supervisord 是的 -n 参数
silent=false                 ; 是否不输出日志，显然这个参数就是启动 supervisord 是的 -s 参数
minfds=1024                  ; 最小文件打开数，对应系统 limit.conf 中的 nofile ,默认最小为1024，最大为4096
minprocs=200                 ; 最小的进程打开数，对应系统的 limit.conf 中的 nproc,默认为 200
;umask=022                   ; process file creation umask; default 022
;user=supervisord            ; 启动 supervisord 服务的用户
;identifier=supervisor       ; supervisord 标识符
;directory=/tmp              ; 服务的工作目录
;nocleanup=true              ; don't clean up tempfiles at start; default false
;childlogdir=/tmp            ; 'AUTO' child log dir, default $TEMP
;environment=KEY="value"     ; key value pairs to add to environment
;strip_ansi=false            ; strip ansi escape codes in logs; def. false


[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface


[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as in [*_http_server] if set
;password=123                ; should be same as in [*_http_server] if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available


;[program:theprogramname]      ; 定义一个进程，[program:xxx]，那么后面就可以通过 xxx 来启动进程
;command=/bin/cat              ; 启动进程时执行的命令，可以是绝对路径、也可以是相对路径
;process_name=%(program_name)s ; 进程名称，默认是 %(program_name)s
;numprocs=1                    ; Supervisor启动这个程序的多个实例，如果numprocs>1，则process_name的表达式必须包含%(process_num)s，默认是1
;directory=/tmp                ; 执行命令时所在的目录
;umask=022                     ; umask for process (default None)
;priority=999                  ; 权重，可以控制程序启动和关闭时的顺序，权重越低：越早启动，越晚关闭。默认值是999
;autostart=true                ; 如果设置为true，当supervisord启动的时候，进程会自动启动
;startsecs=1                   ; 程序启动后等待多长时间后才认为程序启动成功，默认是10秒
;startretries=3                ; supervisord尝试启动一个程序时尝试的次数。默认是3
;autorestart=unexpected        ; 设置为随 supervisord 重启而重启，值可以是false、true、unexpected。false：进程不会自动重启
;exitcodes=0                   ; 一个预期的退出返回码，默认是0
;stopsignal=QUIT               ; 当收到stop请求的时候，发送信号给程序，默认是TERM信号，也可以是 HUP, INT, QUIT, KILL, USR1, or USR2
;stopwaitsecs=10               ; 在操作系统给supervisord发送SIGCHILD信号时等待的时间
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; 如果supervisord以root运行，则会用这里指定的用户启动子程序
;redirect_stderr=true          ; 如果设置为true，进程则会把标准错误输出到supervisord后台的标准输出文件描述符
;stdout_logfile=/a/path        ; 把进程的标准输出写入文件中，如果stdout_logfile没有设置或者设置为AUTO，则supervisor会自动选择一个文件位置
;stdout_logfile_maxbytes=1MB   ; 标准输出log文件达到多少后自动进行轮转，单位是KB、MB、GB。如果设置为0则表示不限制日志文件大小
;stdout_logfile_backups=10     ; 标准输出日志轮转备份的数量，默认是10，如果设置为0，则不备份
;stdout_capture_maxbytes=1MB   ; 当进程处于stderr capture mode模式的时候，写入FIFO队列的最大bytes值，单位可以是KB、MB、GB
;stdout_events_enabled=false   ; 在 stdout 执行写操作时触发事件 PROCESS_LOG_STDOUT
;stdout_syslog=false           ; 将标准输出发送到带有进程名的系统日志中
;stderr_logfile=/a/path        ; 把进程的错误日志输出一个文件中，除非redirect_stderr参数被设置为true
;stderr_logfile_maxbytes=1MB   ; 错误log文件达到多少后自动进行轮转，单位是KB、MB、GB。如果设置为0则表示不限制日志文件大小
;stderr_logfile_backups=10     ; 错误日志轮转备份的数量，默认是10，如果设置为0，则不备份
;stderr_capture_maxbytes=1MB   ; 当进程处于stderr capture mode模式的时候，写入FIFO队列的最大bytes值，单位可以是KB、MB、GB
;stderr_events_enabled=false   ; 在 stderr 执行写操作时触发事件 PROCESS_LOG_STDERR
;stderr_syslog=false           ; 将标准错误输出发送到带有进程名的系统日志中
;environment=A="1",B="2"       ; 一个键值对的list列表
;serverurl=AUTO                ; 是否允许子进程和内部的HTTP服务通讯，如果设置为AUTO，supervisor会自动的构造一个url


;[include]
;files = relative/directory/*.ini ; 可以将配置信息写在多个地方，然后导入进来，类似于 Nginx
```





### 举个栗子

下面我们写一个程序，演示一下：

```
import time 

i = 1
while True:
    with open("1.txt", "a", encoding="utf-8") as f:
        f.write(f"第 {i} 次写入\n")
        time.sleep(2)
    i += 1
```

假设我们这个 py 文件叫做 test.py，位于 /root 下面，那么看看怎么通过 supervisor 启动。首先是编写配置文件，这里叫做 supervisord.conf。

```
[unix_http_server]
file=/var/run/supervisor.sock    

[supervisord]                    
logfile=/var/run/supervisord.log 
pidfile=/var/run/supervisord.pid 

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock 

; 上面这几个文件我们需要单独创建，然后赋予相应的权限即可，因为它们都在 /tmp 下面，会被清理
; 当然你也可以放在别的目录下

; 这里也必须要写上，否则无法使用 supervisorctl
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[program:my_test]             ; 这里是重点，根据自身的情况进行填写
command=python3 test.py       ; 被监控的进程启动命令
directory=/root/              ; 执行命令的所在目录
priority=1                    ; 数字越高，优先级越高
numprocs=1                    ; 启动几个进程
autostart=true                ; 随着supervisord的启动而启动
autorestart=true              ; 自动重启。
startretries=10               ; 启动失败时的最多重试次数
exitcodes=0                   ; 正常退出代码
stopsignal=KILL               ; 用来杀死进程的信号
stopwaitsecs=10               ; 发送SIGKILL前的等待时间
redirect_stderr=true          ; 重定向stderr到stdout
```

此时我们就创建了一个进程：my_test，然后我们启动 supervisord 的时候会自动启动。

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/supervisor_01.png)

我们 supervisord 已经成功启动了，然后我们可以查看状态、重启等等。

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/supervisor_02.png)

我们这里只有一个进程，当然你也可以写任意个进程，进程名字叫做 my_test。如果你需要中途添加别的进程，那么修改完配置文件之后通过 supervisorctl reload 重启即可，或者使用 supervisorctl update。

我们也可以通过页面来进行查看，只不过需要设置 `[inet_http_server]`，但是这样做的话可能导致 CPU 和 内存占用率增高，一般不设置。





### supervisord 设置为开机自启

然后我们需要将 supervisord 设置为开机自启，在目录/usr/lib/systemd/system/ 新建文件supervisord.service，并添加配置内容：

```
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service nss-user-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/bin/supervisord -c /root/supervisord.conf
ExecStop=/usr/local/bin/supervisord shutdown
ExecReload=/usr/local/bin/supervisord reload
killMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

```

执行 systemctl enable supervisord 启动服务，然后执行 systemctl is-enabled supervisord 验证是否开启自己。