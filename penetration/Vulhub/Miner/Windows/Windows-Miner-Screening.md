以下是详细版流程，适用于初步排查病毒但没有探查到的情况，进而需要全面排查。而初步排查只需要每个步骤执行开始的一小部分即可。

## 排查思路

1、首先查找可疑用户，记录好可疑信息。

```bash
#####用户相关命令
#1、查找可疑用户
cmd->lusrmgr.msc
net user
wmic useraccount get name,sid,status
```

2、利用netstat 查看可疑端口及异常连接，然后查看可疑端口对应的pid，进行pid搜查。

```bash
#####端口相关命令
#netstat
netstat -ano
```

3、没有process explore时，只能用命令行查看，不实用，最好用process explore查看。

```bash
#####进程相关命令
#1、tasklist
#查看所有进程及其CPU总计使用时间
tasklist /v 
#查看进程及其对应的服务和PID
tasklist /svc
#查看某个进程的可执行路径
wmic process get Caption,CommandLine,ProcessId /value
#2、powershell
#查看所有进程及CPU总计使用时间
get-process
#3、cmd
cms->msinfo32->软件环境->正在运行的服务
```

4、登录、开机启动项，以及调试启动项

比较特别，涵盖了注册表、快捷启动项、组策略、startup apps、msfconfig。

```bash
###启动项相关
#1、注册表
#当前登录用户的启动项
HKEY_CURRENT_USER\Software\Micorsoft\Windows\Currentversion\Run
#所有用户公共的启动项
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
#仅第一次登录时运行一次的启动项
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runonce

#2、快捷启动项
#当前登录用户的启动项
shell:startup
#所有用户公共的启动项
shell:common startup

#3、startup apps和msfconfig和任务管理器中的启动
#它们均为当前用户和所有用户公共启动项的和（注册表+快捷启动项）
#在win7后msfconfig重定向到任务管理器启动中
cmd->msfconfig
开始菜单->startup apps

#4、组策略
#域环境中由域管理员设置的登录启动项
cmd->gpedit.msc


###其中快捷启动项和注册表的并集为其某一类启动项的所有启动项

```

```bash
#windows中有个比较特别的注册表项，用作调试
#当应用启动时会首先检查该注册表项，如果其该注册表项的debugger键值被设置为非空，则会启动debugger键值设置的路径文件而不会启动原应用
#HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File ExecutionOption
```

5、cmd->taskschd.msc查看计划任务

6、在/etc/rc.d/rc.local、chkconfig --list（Red Hat系）、systemctl list-unit-files --type=service --state=enabled（Debian系）等任何自启工具中检查恶意服务自启脚本，这是对于rc初始化检查的补充检查，服务既可能在rc初始化时就被启动了，也可能是在初始化完成后利用这些工具进行启动。

**init搜查**：在/etc/rc.d/rc.local中检查脚本中可疑的初始化后运行的服务和命令（同rc搜查，如果是pid搜查拓展的init搜查，就在文件名或文件内容中搜恶意进程名，但也可能由于脚本跳板而搜不到），观察是否执行了可疑程序和命令。然后再同样对chkconfig和systemd等其它工具托管的自启服务进行搜查，还有源码包。这是另一种隐藏运行恶意文件的方法，需要对其进行记录并删除相关文件。

```bash
#####服务相关命令
#cmd
cmd->services.msc

#查看windows服务对应的端口
%system%/system32/drivers/etc/services
```

7、粘滞键排查。

连续点击5次shift或者ctrl等特殊键，出现粘滞键提示，攻击者可能会利用调试注册表项debugger更改粘滞键的执行路径，从而通过粘滞键留下后门。

```bash
#####粘滞键相关
#粘滞键原程序位置
C:\Windows\System32\sethc.exe

#调试注册表项
#HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File ExecutionOption
```

8、在cmd->eventvwr.msc查看所有日志。

security日志主要记录和安全相关的日志，包括登录尝试、资源访问、策略更改、权限使用等。 

```bash
###一般首先要排查的就是security的4624、4625事件ID查看是否存在爆破的情况并进行溯源
###Security日志事件ID
#4624登录成功
#4625登录失败
#4720创建用户
#4672以管理员身份登录
#4274修改用户密码
#4634注销成功
#4647用户启动的注销
#4738已更改用户账户（更改指更改了用户的信息，如重置密码、用户登录名）
#4776成功/失败的账户认证
#6005，6006，6009表示不同状态的机器的情况
#6005表示EventLog事件日志服务已启动
#6006表示EventLog事件日志服务已停止
#6009表示EventLog按crtl、alt、delete键关机（非正常关机）
```

system日志主要记录与操作系统和系统服务相关的事件，包括驱动程序加载、硬件故障、系统组件错误和服务状态变化。

application日志主要记录由各个应用程序和软件生成的事件，包括应用程序错误、警告和信息性事件等。

## 清理思路

在找到病毒核心后（对计算机造成实际危害的核心程序）

1、先将病毒核心保存一份样本，并将其作为