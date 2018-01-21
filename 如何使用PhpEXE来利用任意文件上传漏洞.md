## 如何使用PhpEXE来利用任意文件上传漏洞
任意文件上传在Web应用程序中是非常常见的，可能会被滥用来上传恶意文件，然后危害服务器。通常，攻击者将根据支持的任何服务器端编程语言来选择一个有效载荷。因此，如果易受攻击的应用程序在PHP中，那么显然PHP是支持的，因此一个简单的选择就是使用诸如Metasploit的PHP meterpreter之类的PHP有效载荷。然而，PHP meterpreter并不像Windows meterpreter那样共享相同的性能。所以在现实中，会发生什么呢？你可能会想升级到一个更好的shell，在这个过程中需要额外的手动工作。
那么对于这种类型的场景为什么限制你的有效载荷选项 ，你应该使用PhpEXE mixin。它用作PHP中的有效负载，将最终的恶意可执行文件写入远程文件系统，然后在使用后自行清除，因此不会留下任何痕迹。

### 要求
要使用PhpEXEmixin，应该满足一些典型的可利用的要求：
* 您必须在Web服务器上找到可写的位置。
* 同一个可写位置也应该可以通过HTTP请求读取。
注意:对于任意文件上传漏洞，通常有一个目录包含上传的文件，并且是可读的。如果这个bug是由于目录遍历造成的，那么临时文件夹(来自操作系统或者web应用程序)就是你的选择。

### 用法
* 首先在您的Metasploit3类范围内包含mixin ，如下所示
~~~
include Msf::Exploit::PhpEXE
~~~
* 使用php生成载荷(php stager) `get_write_exec_payload`
~~~
p = get_write_exec_payload
~~~
* 如果您正在使用Linux目标，则可以将其设置unlink_self为true，这将自动清除可执行文件：
~~~
p = get_write_exec_payload(:unlink_self=>true)
~~~
在Windows上，您可能无法清除可执行文件，因为它可能仍在使用中。如果无法自动清除恶意文件，则应始终警告用户，以便在渗透测试期间手动完成。
* 上传有效载荷
这个时候你可以上传     `get_write_exec_payload`生成的有效载荷然后使用GET请求来调用它。如果您不知道如何发送GET请求，请参考以下文章
https://github.com/rapid7/metasploit-framework/wiki/How-to-Send-an-HTTP-Request-Using-HTTPClient
