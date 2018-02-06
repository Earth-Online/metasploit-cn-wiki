Meterpreter一直需要进行配置，以便它知道如何与Metasploit通信。多年来，这种配置管理是通过metsrv使用简单的“字符串替换”方法热修补DLL /二进制文件的副本来实现的。这很好地支持了一些情况，但是限制了Meterpreter的灵活性以及对处理多个传输的支持。
这不仅仅是传输系统被锁定，而是提供载荷的能力比核心Meterpreter（metsrv）本身更重要。将其他形式的信息传递给Meterpreter实例也不是一件容易的事，因为stagers只能传入活动套接字句柄的副本。
对Meterpreter的最近的修改已经取消了这个旧的方法，取而代之的是一个动态的配置块，可以用来缓解这些问题，并为其他更有趣的事情提供了灵活性。

本文档包含有关新配置块的结构和布局的信息，以及Meterpreter如何使用它。

## 如何找到配置
在过去，Meterpreter已经要求stager（或者stage0 一些喜欢叫它）将句柄传递给活动套接字，以便它可以在不创建新套接字的情况下（至少在TCP连接的情况下）接管通信。虽然这个功能仍然是必需的，但它不会像以前那样发生。相反，Meterpreter现在要求stager传递一个指向配置块开始的指针。只要内存区域被标记为`RWX`，配置块可以在内存中的任何地方。

### 在Windows Meterpreter中加载配置
加载Windows Meterpreter的第1阶段现在使用一个名为`meterpreter_loader` ([Win x86](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/payload/windows/meterpreter_loader.rb), [Win x64](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/payload/windows/x64/meterpreter_loader.rb)),的新载入程序，它执行以下操作
* 从磁盘加载`metsrc`DLL。
* 修补DLL的DOS头，以便它包含可执行的shellcode，正确地初始化metsrv并计算指向metsrv内存结尾的位置。它还可以使用任何现有的套接字值（在edi或rdi根据体系结构中找到），并直接写入配置（稍后会详细介绍）。
* 生成一个配置块并将其附加到metsrv二进制文件。

其结果是一旦准备好了有效载荷就具有以下结构：

```
  +--------------+
  | Patched DOS  |
  |    header    |
  +--------------+
  |              |
  .              .
  .  metsrv dll  .
  .              .
  |              |
  +--------------+
  | config block |
  +--------------+
```

### 在POSIX Meterpreter中加载配置
POSIX Meterpreter以相同的方式工作，除了它没有引导的补丁头。有效载荷的格式是相同的，因此看起来像这样：


```
  +--------------+
  |              |
  .              .
  .  metsrv bin  .
  .              .
  |              |
  +--------------+
  | config block |
  +--------------+
```

**TODO：@bcook-r7确认这是正确的

## 配置块结构
为了将信息传递给Meterpreter，而不是中断，需要一个已知的配置格式。这种格式需要在每次调用时保持一致，就像您期望的任何配置一样。在二进制Meterpreter（POSIX和Windows）的情况下，该配置块包含以下内容：
* 一个会话配置块。
* 一个或多个传输配置块，后跟一个终止符。
* 一个或多个扩展配置块，后跟一个终止符。

下面将详细介绍这些模块。

### 会话配置块
会话配置块的概念用于包装以下值：
* 套接字句柄 - 当通过TCP通信调用Meterpreter时，活动套接字已经在使用中。这个套接字句柄在metsrv执行时被Meterpreter重复使用。这个套接字句柄由加载程序实时写入配置块。它存储在会话配置块中，以便它具有已知的位置。即使在64位平台上，此值也始终是32位DWORD。
* 退出功能 - 该值是一个32位的DWORD值，标识在终止Meterpreter会话时应该使用的方法。这个值相当于表示被调用函数的Block API Hash。Meterpreter用来把处理这个的责任委托给调用它的stager。Meterpreter不再执行此操作，而是它自己处理Meterpreter会话的关闭，因此必须在配置中使用所选的终止方法。
* 会话到期值 - 这是一个32位的DWORD，它包含Meterpreter会话应持续的秒数。当Meterpreter正在运行时，会持续检查此值，如果达到会话到期时间，则Meterpreter会自行关闭。有关更多信息，请阅读Timeout文档
* UUID - 这是一个16字节的值，表示有效负载UUID。UUID是Metasploit的一个新概念，目的是追踪有效载荷类型和来源，并确定Metasploit收到的会话是供当前安装使用的。有关更多信息，请阅读UUID文档

这块在内存中的布局如下所示：

```
  +--------------+
  |Socket Handle |
  +--------------+
  |  Exit func   |
  +--------------+
  |Session Expiry|
  +--------------+
  |              |
  |     UUID     |
  |              |
  |              |
  +--------------+

  | <- 4 bytes ->|
```

有了这个结构，Meterpreter知道会话配置块的大小正好是28字节。

会话配置块描述可以在Meterpreter[源文件](https://github.com/rapid7/meterpreter/blob/master/source/common/config.h#L25).中找到。

### 传输配置块

传输配置块是一个术语，用于指代有效负载中存在的一组传输配置。Meterpreter现在支持多个传输，所以配置也应该支持多个传输。

处理运输配置有两个主要问题：

1. 该配置应该允许指定许多传输配置。
2. 配置应该允许每个运输是不同的类型和大小。
Meterpreter目前的运输实施提供了两个主要的“类”的运输，那些是HTTP(S)和TCP。这些传输类中的每一个都需要不同的配置值以及常用的值才能正常工作。

#### 通用配置值
这两个HTTP(S)和TCP传输共同的值是：
* URL - 此值是传输的元描述，不仅用作传输本身的配置元素，而且还用作确定此块代表何种传输类型的方式。该字段总共是512 字符（Windows Meterpreter使用wchar_t，而POSIX Meterpreter使用char）。传输类型由URL中的scheme元素指定，URL的主体指定关键信息（如主机和端口信息）。Meterpreter检查这个来决定使用什么类型的传输块，因此能够确定块的大小。有效值如下所示：
    * `tcp://<host>:<port>` - 表示此有效负载是反向 IPv4 TCP连接
    * `tcp6://<host>:<port>?<scope>`-  表示此有效负载是反向 IPv6 TCP连接。
    * `tcp://:<port>` - - 表示这个有效载荷是在指定的端口上监听的绑定有效载荷（注意没有指定主机）。
    *  `http://<host>:<port>/<uri>` - 表示这个有效载荷是一个HTTP连接（只能是反向的）。
    *  `https://<host>:<port>/<uri>` -表示这个有效载荷是一个HTTPS连接（只能是反向的）。
* Communications expiry - 此值是另一个32位DWORD值，表示成功的数据包/接收呼叫之间等待的秒数。有关更多信息，请阅读Timeout文档
* Retry total - 此值是32位DWORD值，表示Meterpreter在放弃之前应继续尝试重新连接此传输的秒数。有关更多信息，请阅读Timeout文档
* Retry wait - 此值是32位DWORD值，表示Meterpreter在此传输上重新连接的每次尝试之间的秒数。有关更多信息，请阅读Timeout文档（即将推出的链接）。

该块在内存中的布局如下所示：


```
  +--------------+
  |              |
  |      URL     |
  .              .
  .              .  512 characters worth
  .              .  (POSIX -> ASCII -> char)
  .              .  (Windows -> wide char -> wchar_t)
  .              .
  |              |
  +--------------+
  |  Comms T/O   |
  +--------------+
  |  Retry Total |
  +--------------+
  |  Retry Wait  |
  +--------------+

  | <- 4 bytes ->|
```

常见的传输配置块描述可以在Meterpreter[源文件](https://github.com/rapid7/meterpreter/blob/master/source/common/config.h#L33)找到

####　TCP配置值
此时，没有TCP特定的配置值，因为通用配置块满足所有的TCP传输需要。这可能会改变。

#### HTTP/S配置值
HTTP并且HTTPS连接具有许多额外的配置值，这些配置值是为了使其在各种环境中正常运行所必需的。这些值是：

* 代理主机 - 在需要手动设置代理的环境中，此字段包含要使用的代理的详细信息。该字段的大小是128字符（只有wchar_t，HTTP/S在POSIX中我们还没有传输）并且可以采用以下格式之一：
    * `http://<proxy ip>:<proxy port>` 在HTTP代理的情况下。
    * `socks=<socks ip>:<sock port>` 在socks代理的情况下。
* 代理用户名 - 某些代理需要认证。在这种情况下，这个值包含了用来验证给定代理的用户名。这个字段的大小是64个字符（wchar_t）。
* 代理密码 - 在需要代理验证的情况下，该值将伴随用户名字段。它包含用于对代理进行身份验证的密码，大小也是64个字符（wchar_t）。
* 用户代理字符串 - 可定制的用户代理字符串。这会更改在HTTP/S向Metasploit发出请求时使用的用户代理。这个字段的大小是256个字符（wchar_t）。
* 预期的SSL证书散列 - Meterpreter能够验证Metasploit在使用HTTPS时提供的SSL证书。此值包含20字节预期证书的SHA1散列。有关更多信息，请阅读SSL证书验证文档（链接即将推出）。

上面显示的所有值都需要在配置中指定，包括用于普通HTTP连接的SSL证书验证。不使用的值应该被清零。

HTTP/S配置的结构如下。
```
  +--------------+
  |              |
  |  Proxy host  |
  .              .  128 characters worth (wchar_t)
  |              |
  +--------------+
  |              |
  |  Proxy user  |
  .              .  64 characters worth (wchar_t)
  |              |
  +--------------+
  |              |
  |  Proxy pass  |
  .              .  64 characters worth (wchar_t)
  |              |
  +--------------+
  |              |
  |  User agent  |
  .              .  256 characters worth (wchar_t)
  |              |
  +--------------+
  |              |
  |   SSL cert   |
  |   SHA1 hash  |
  |              |
  |              |
  +--------------+

  | <- 4 bytes ->|
```
HTTP/S传输配置块描述可以在[Meterpreter source](https://github.com/rapid7/meterpreter/blob/master/source/common/config.h#L48).找到

#### 传输配置列表
如前所述，可以指定多个这些传输配置块。为了便于操作，Meterpreter需要知道传输“列表”何时结束。使用`URL` Meterpreter可以确定块的大小，并可以根据发现的类型移动到下一个块。只要Meterpreter检测到URL字符串长度为零的传输配置值（即NULLPOSIX中的单个ASCII字符和NULLWindows中的单个多字节字符），它将假定传输列表已终止。紧随其后的字节被认为是扩展配置的开始，这在下一节中进行了介绍。

### 扩展配置块
扩展配置块旨在允许Meterpreter有效载荷包含用户想要捆绑的任何额外的扩展。目标是提供具有Stageless有效载荷（即将推出）的能力，并提供在扩展期间共享扩展的手段迁移（尽管这还没有实施）。每个扩展必须已经用Reflective DLL Injection支持进行编译，因为这是在Meterpreter启动时用于加载扩展的机制。有关此设施的更多信息，请参阅Stageless有效载荷（即将推出）文档。

该扩展配置块也作为“列表”，以允许包含任意数量的扩展。每个扩展条目都需要包含以下内容：
* 大小 - 这是扩展DLL本身的确切大小（以字节为单位）。该值是一个32位的DWORD。
* 扩展二进制 - 这是从DLL直接复制的完整二进制文件。该值需要与“大小”字段中指定的长度完全相同。

从配置中加载扩展时，Meterpreter将继续解析条目，直到找到size值0。此时，Meterpreter假定它已经到达扩展列表的末尾，并停止解析。

结构布局如下：

```
  +--------------+
  |  Ext. Size   |
  +--------------+
  | Ext. content |
  +--------------+
  |  NULL term.  |
  |   (4 bytes)  |
  +--------------+
```

## 配置块概述
总而言之，以下显示了完整配置的布局：
```
+--------------+
  |Socket Handle |
  +--------------+
  |  Exit func   |
  +--------------+
  |Session Expiry|
  +--------------+
  |              |
  |     UUID     |
  |              |
  |              |
  +--------------+
  |  Transport 1 |
  |  tcp://...   |
  .              .
  |              |
  +--------------+
  |  Comms T/O   |
  +--------------+
  |  Retry Total |
  +--------------+
  |  Retry Wait  |
  +--------------+
  |  Transport 2 |
  |  http://...  |
  .              .
  |              |
  +--------------+
  |  Comms T/O   |
  +--------------+
  |  Retry Total |
  +--------------+
  |  Retry Wait  |
  +--------------+
  |              |
  |  Proxy host  |
  |              |
  +--------------+
  |              |
  |  Proxy user  |
     |              |
  +--------------+
  |              |
  |  Proxy pass  |
  |              |
  +--------------+
  |              |
  |  User agent  |
  |              |
  +--------------+
  |              |
  |   SSL cert   |
  |   SHA1 hash  |
  |              |
  +--------------+
  |  NULL term.  |
  |(1 or 2 bytes)|
  +--------------+
  | Ext 1. Size  |
  +--------------+
  |Ext 1. content|
  +--------------+
  | Ext 2. Size  |
  +--------------+
  |Ext 2. content|
  +--------------+
  |  NULL term.  |
  +--------------+

```