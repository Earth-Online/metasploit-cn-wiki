在Metasploit模块中使用多个网络mixin总是一件棘手的事情，因为很可能会碰到重叠的数据存储选项，变量，方法等问题.超级调用仅适用于一个mixin等。这被认为是高级的模块开发，有时可能是相当痛苦地弄清自己的。为了改善Metasploit的开发体验，我们举几个例子来演示常见的场景，需要使用多个mixin来实现开发。

### 今天的课程：发送HTTP请求来攻击目标机器，并使用HttpServer来传送负载。
假设您想要利用Web服务器或Web应用程序。你可以代码执行，但你需要找到一种方式来提供最终的有效载荷（可能是一个可执行文件），而HTTP服务器恰好是你的选择。
这里是你如何设置它：
```ruby

##
# This module requires Metasploit: http://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

require 'msf/core'

class MetasploitModule < Msf::Exploit::Remote
  Rank = NormalRanking

  include Msf::Exploit::Remote::HttpClient
  include Msf::Exploit::Remote::HttpServer::HTML

  def initialize(info={})
    super(update_info(info,
      'Name'           => "HttpClient and HttpServer Example",
      'Description'    => %q{
        This demonstrates how to use two mixins (HttpClient and HttpServer) at the same time,
        but this allows the HttpServer to terminate after a delay.
      },
      'License'        => MSF_LICENSE,
      'Author'         => [ 'sinn3r' ],
      'References'     =>
        [
          ['URL', 'http://metasploit.com']
        ],
      'Payload'        => { 'BadChars' => "\x00" },
      'Platform'       => 'win',
      'Targets'        =>
        [
          [ 'Automatic', {} ],
        ],
      'Privileged'     => false,
      'DisclosureDate' => "Dec 09 2013",
      'DefaultTarget'  => 0))

      register_options(
        [
          OptString.new('TARGETURI', [true, 'The path to some web application', '/']),
          OptInt.new('HTTPDELAY',    [false, 'Number of seconds the web server will wait before termination', 10])
        ], self.class)
  end

  def on_request_uri(cli, req)
    print_status("#{peer} - Payload request received: #{req.uri}")
    send_response(cli, 'You get this, I own you')
  end

  def primer
    print_status("Sending a malicious request to #{target_uri.path}")
    send_request_cgi({'uri'=>normalize_uri(target_uri.path)})
  end

  def exploit
    begin
      Timeout.timeout(datastore['HTTPDELAY']) { super }
    rescue Timeout::Error
      # When the server stops due to our timeout, this is raised
    end
  end
end
```
以下是运行上述示例时发生的情况：

1.封装在Timeout块的超级调用将启动Web服务器。
2.在Web服务器处于无限循环状态之前，会调用primer()方法,这是您发送恶意请求以获取代码执行的地方。
3.您的HttpServer根据请求提供最终的有效载荷
4.10秒后，模块引发超时异常。Web服务器终止。

如果你想知道为什么Web服务器必须在一段时间后终止，这是因为如果模块无法在目标机器上执行代码执行，显然它永远不会询问你的Web服务器的恶意负载，所以没有意义永远保持活动.通常情况下，获得有效载荷请求也不需要很长时间，所以我们保持了超时。
上例的输出应该如下所示：

```
msf exploit(test) > run
[*] Exploit running as background job.

[*] Started reverse handler on 10.0.1.76:4444 
[*] Using URL: http://0.0.0.0:8080/SUuv1qjZbCibL80
[*]  Local IP: http://10.0.1.76:8080/SUuv1qjZbCibL80
[*] Server started.
[*] Sending a malicious request to /
msf exploit(test) >
[*] 10.0.1.76        test - 10.0.1.76:8181 - Payload request received: /SUuv1qjZbCibL80
[*] Server stopped.

msf exploit(test) >
```

### 相关文章：

* https://github.com/rapid7/metasploit-framework/wiki/How-to-Send-an-HTTP-Request-Using-HTTPClient
* https://github.com/rapid7/metasploit-framework/wiki/How-to-write-a-browser-exploit-using-HttpServer
* https://community.rapid7.com/community/metasploit/blog/2012/12/17/metasploit-hooks
