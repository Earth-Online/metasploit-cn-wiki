## 如何开始写一个post模块
post模块开发对您的编程技能是一个挑战。这不像写一个基于内存损坏的攻击，从技术上说，通常是制造恶意输入 - 一个字符串。post模块更多的是关于正确的模块设计，Ruby和Metasploit库的实用知识。这也是一个非常有价值的技能，因为如果在弹出一个shell之后你不知道该怎么做，渗透测试的重点是什么？ 另外，如果一个模块不工作？你是否愿意等待几天，几周甚至几个月的时间让其他人为你解决？可能不会。如果你自己知道该怎么做，那么你可以更早地修复它，继续你的渗透，做更多的事情。所以学习post模块开发！这对你和你的事业都有好处

### 计划你的模块
就像编写一个软件一样，在你开始写代码之前，你应该有一个清晰明确的目标，就是你的post模块做什么。在单个模块中具有多个功能从来就不是一个好主意。
例如:盗取网络配置文件，窃取密码，哈希值，shell历史记录等。相反，您应该将其分解为多个模块。
您还应该考虑要支持的会话类型：meterpreter或shell。理想情况下，两者都支持。但是如果你必须在两者之间进行选择，那么在Windows上你应该选择Windows Meterpreter。在Linux上，shell会话类型比Linux Meterpreter更为强大，但希望在不久的将来会改变。对于没有Meterpreter的平台，显然你唯一的选择是一个shell。
另一个重要的事情是考虑你的模块如何在不同的发行版/系统上执行。例如你想在linux运行一个`ifconfig`命令.在ubuntu只需要简单的运行`ifconfig`就可以了.但是在不同的linux发行版可能不知道你在做什么.所以你必须更具体一些，而不是直接`/sbin/ifconfig`。
是`C:\WINDOWS\`还是`C:\WinNT`?这是两个,是不是`C:\Documents and Settings\[User name]`或者`C:\Users\[User name]`.都取决与window版本.更好的解决方案是使用环境变量
总是做你的准备,，并包含你可以想到的场景。而最重要的是，让你的虚拟机测试

### post模块的类别
post模块根据其行为进行分类。例如，如果收集数据，自然会进入`gather`类别。如果它添加/更新/或删除用户，它属于`manage`。这里有一个列表作为参考：

| 类别   |  描述  |
| --- | --- |
|  gather  |  涉及数据收集/收集/枚举的模块。  |
|   gather/credentials |   窃取凭据的模块。 |
|   gather/forensics | 涉及取证数据收集的模块。   |
| manage   |   修改/操作/操作系统上的某些东西的模块。会话管理相关的任务，如迁移，注入也在这里。 |
|  recon  |   这些模块将帮助您在侦察方面了解更多的系统信息，而不是关于数据窃取。了解这与`gather`类型模块不一样。 |
|   wlan |  用于WLAN相关任务的模块。  |
|  escalate  |   这是不赞成的，但由于受欢迎，模块仍然在那里。这曾经是特权升级模块的地方。所有权限升级模块不再被视为后期模块，它们现在是exploit。 |
|   capture | 涉及监控数据收集的模块。例如：密钥记录。  |


### 会话对象
所以你知道魔戒怎么样，人们完全沉迷于一环？那么,这就是会话对象,你不能没有的一个对象.所有的post模块和其他相关的mixin基本上都是建立在会话对象之上的，因为它知道被入侵主机的所有信息，并允许你命令它。

您可以使用该`session`方法来访问会话对象或其别名`client`。与irb交互的最佳方式是通过`irb`命令，以下是一个例子：
~~~
msf exploit(handler) > run

[*] Started reverse handler on 192.168.1.64:4444 
[*] Starting the payload handler...
[*] Sending stage (769536 bytes) to 192.168.1.106
[*] Meterpreter session 1 opened (192.168.1.64:4444 -> 192.168.1.106:55157) at 2014-07-31 17:59:36 -0500

meterpreter > irb
[*] Starting IRB shell
[*] The 'client' variable holds the meterpreter client

>> session.class
=> Msf::Sessions::Meterpreter_x86_Win
~~~
在这一点上，你有权力使用他们。但请注意，上面的例子是一个Msf::Sessions::Meterpreter_x86_Win对象。实际上还有几个不同的东西：command_shell.rb，meterpreter_php.rb，meterpreter_java.rb，meterpreter_x86_linux.rb等等。每个行为都有所不同，所以实际上很难解释它们，但是它们是在lib/msf/base/sessions/ 目录下,所以你是可以看到它们是怎么工作的,或者你可以试一下因为你已经在irb命令行了.
在ruby有两个方便调试对象目的的对象方法.第一个是`methods`,它将会列出对象中全部公开和受防护的方法.
~~~
session.methods
~~~
另一个是inspect，它返回一个对象的人类可读的字符串：
~~~
session.inspect
~~~
您还可以查看[其他当前的post模块](https://github.com/rapid7/metasploit-framework/tree/master/modules/post)，并查看他们如何使用其会话对象。

### Msf::Post Mixin
正如我们所解释的，大多数post模块mixin是建立在会话对象之上的，而且它还有很多。但是，有一个你显然不能没有的一个`Msf::Post `.mixin。当你用这个mixin创建一个post模块时，很多其他的mixin也已经包含在各种场景中，更具体一些：
* msf/core/post/common -  post模块使用的通用方法，例如`cmd_exec`
* msf/core/post_mixin - 跟踪会话状态。
*  msf/core/post/file - 文件系统相关的方法
*  msf/core/post/webrtc - 使用WebRTC与目标机器的摄像头进行交互
*  msf/core/post/linux - 通常不是很多在这, 只是特定与linux的`get_sysinfo` 和 `is_root`? 
*  msf/core/post/osx - `get_sysinfo`，`get_users`，`get_system_accounts`，get_groups和用于操作的目标计算机的网络摄像头的方法。
*  msf/core/post/solaris - 非常像linux mixin。相同的方法，但是是对于Solaris。
*  msf/core/post/unix - get_users get_groups enum_user_directories
*  msf/core/post/windows -大部分的开发时间都花在这里。Windows帐户管理，事件日志，文件信息，Railgun，LDAP，netapi，powershell，注册表，wmic，服务等。

### 模板
在这里我们有一个post模块模板 正如你所看到的，有一些需要填写的必填字段。我们将解释每个:
~~~
##
# This module requires Metasploit: http://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

require 'msf/core'

class MetasploitModule < Msf::Post

  def initialize(info={})
    super(update_info(info,
        'Name'          => '[Platform] [Module Category] [Software] [Function]',
        'Description'   => %q{
          Say something that the user might want to know.
        },
        'License'       => MSF_LICENSE,
        'Author'        => [ 'Name' ],
        'Platform'      => [ 'win', 'linux', 'osx', 'unix', 'bsd' ],
        'SessionTypes'  => [ 'meterpreter', 'shell' ]
    ))
  end

  def run
    # Main method
  end

end

~~~
`Name`字段应该以平台开头,就像Multi, Windows, Linux, OS X.接下来是模块类别,例如Gather, Manage, Recon, Capture, Wlan..接下来是软件的名字,最后几个单词描述这个模块的功能,例子:"Multi Gather RndFTP Credential Enumeration".
在`Description`字段应该解释模块做什么，什么事情要留意，具体要求，多多益善。目标是让用户了解他所使用的内容，而不需要实际读取模块的源代码并找出结果。相信我，他们中的大多数人不会。
`Author `字段是你的名字。格式应该是“名称”。如果你想在那里有你的Twitter，留下它作为一个评论，例如:“名称＃handle”
该`Platform`字段表示所支持的平台，例如：win, linux, osx, unix, bsd.
这个`Session type`字段应该是meterpreter或者shell,你应该尝试支持更多
最后，这个`run`方法就像你的main方法。开始在那里写你的代码

### 基本的git命令
> 和如何开始写exploit的一样