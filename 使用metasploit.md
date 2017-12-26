### 从这里开始
*  https://www.offensive-security.com/metasploit-unleashed/
* https://community.rapid7.com/community/metasploit/
* https://github.com/rapid7/metasploit-framework/wiki/Evading-Anti-Virus

### 数据库问题
如果数据库没有自动连接，请确保它正在运行：

Linux：`$ netstat -lnt | grep 7337` 其中7337是您在安装期间设置的端口
Windows：在任务管理器中查找postgres.exe进程。

如果postgres没有运行，请尝试手动启动它：

Linux：`$ sudo /etc/init.d/metasploit start` 或者如果你没有选择作为服务安装：`$ sudo /opt/metasploit*/ctlscript.sh start`
Windows：`Start -> Metasploit -> Services -> Start Services`

一旦postgres运行并监听，请回到msfconsole：
~~~
msf > db_connect
~~~