## 如何使用filedropper清理文件
在某些exploit场景中，例如本地特权升级，命令执行，写入权限攻击，SQL注入等，很有可能必须上传一个或多个恶意文件才能获得目标机器的控制权。那么聪明的攻击者不应该放任何东西，所以如果一个模块需要在文件系统上放一些东西，那么在目标服务之后立即删除它是很重要的。这就是为什么我们创建了FileDropper mixin。

### 例子
FileDropper mixin是一个文件管理器，允许您跟踪文件，然后在创建会话时将其删除。要使用它，首先要包含mixin：
~~~
include Msf::Exploit::FileDropper
~~~
接下来，通过使用`register_file_for_cleanup`方法创建一个会话之后，告诉FileDropper mixin文件将在哪里。每个文件名都应该是完整路径，或相对于当前的会话工作目录。例如，如果我想要将有效负载上载到目标机器的远程路径C:\Windows\System32\payload.exe，那么我的声明可以是：
~~~
register_file_for_cleanup("C:\\Windows\\System32\\payload.exe")
~~~
如果我的会话的当前目录已经在Ｃ:\Windows\System\那么我可以简单地做：
~~~
register_file_for_cleanup("payload.exe")
~~~
如果你想注册多个文件，你也可以提供文件名作为参数：
~~~
register_file_for_cleanup("file_1.vbs", "file_2.exe", "file_1.conf")
~~~
请注意，如果您的漏洞利用模块使用on_new_session，您实际上覆盖了FileDropper的on_new_session。