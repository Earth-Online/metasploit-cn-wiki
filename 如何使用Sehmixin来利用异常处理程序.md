异常处理程序覆盖曾经是利用堆栈缓冲区溢出的一种非常流行的技术，但在新程序中不再那么常见，因为它们很可能是用SafeSEH编译的。有一点，即使在启用了SafeSEH的情况下，仍然有可能通过堆喷来滥用异常处理程序，但是当然，内存保护并不止于此。DEP/ASLR最终来拯救，所以几乎结束了SEH漏洞的辉煌岁月。您可能仍然可以找到不利用SafeSEH编译的易受攻击的应用程序，但是该应用程序可能已经过时，不再维护，或者它更像是开发人员的学习实验。哦，这可能已经是一个漏洞了。尽管如此，利用异常处理来利用堆栈缓冲区溢出还是有趣的，所以如果你遇到它，使用`Seh` mixin

### 要求
为了能够使用SEH mixin，必须满足一些可利用的需求：
* 易受攻击的程序没有SafeSEH
* 没有DEP（数据执行保护）。mixin使用短暂的跳转来执行有效载荷，这意味着内存必须是可执行的。正如名字所暗示的，DEP阻止了这一点。

### 示例
首先，确保你在你的模块的`Metasploit3`类的范围内包含了 `Seh` mixin 
~~~
include Msf::Exploit::Seh
~~~

接下来，您需要`Ret`来为SE处理程序设置一个地址。这个地址应该放在你的模块的元数据中，具体在下面的`Targets`。在Metasploit中，每个目标实际上是一个由两个元素组成的数组。第一个元素只是目标的名称（目前没有严格的命名风格），第二个元素实际上是一个字典，其中包含特定于该目标的信息，例如目标地址。以下是设置Ret地址的示例：

~~~
'Targets'        =>
  [
    [ 'Windows XP', {'Ret' => 0x75022ac4 } ] # p/p/r in ws2help.dll
  ]
~~~

正如你所看到的，记录Ret地址的作用以及指向那个DLL 也是一个好习惯。
Ret实际上是一种特殊的key，因为它可以通过`target.ret`在模块中使用。在我们的下一个例子中，你会看到`target.ret`被用来代替原始目标地址的编码。
如果您需要一个工具来为ret地址 查找POP/POP/RET,你能使用metasploit的`msfbinscan`工具,它位于tools目录下.
好的，现在我们来看看这些方法。`Seh` mixin 提供了两种方法：
* `generate_seh_payload` - 生成一个虚假的SEH记录，并在之后附上有效载荷。这是一个例子：
~~~
buffer = ''
buffer << "A" * 1024 # 1024 bytes of padding
buffer << generate_seh_payload(target.ret) # SE record overwritten after 1024 bytes
~~~
buffer内存中的实际布局应该是这样的：
~~~
[ 1024 bytes of 'A' ][ A short jump ][ target.ret ][ Payload ]
~~~

* `generate_seh_record` -  在没有有效载荷的情况下生成假SEH记录，以防您想将有效载荷放置在其他地方。代码示例：
~~~
buffer = ''
buffer << "A" * 1024 # 1024 bytes of padding
buffer << generate_seh_payload(target.ret)
buffer << "B" * 1024 # More padding
~~~

内存布局应该是这样的：
~~~
[ 1024 bytes of 'A' ][ A short jump ][ target.ret ][ Padding ]
~~~

### 参考

https://www.corelan.be/index.php/2009/07/25/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-3-seh/

https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/exploitation/seh.rb

https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/seh.rb