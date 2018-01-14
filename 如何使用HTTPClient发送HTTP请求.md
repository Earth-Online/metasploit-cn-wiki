## 如何使用HTTPClient发送HTTP请求
这是一个如何编写一个使用HttpClient mixin发送基本HTTP请求的模块的例子。

### 主要有两种常见的方法：
* send_request_raw - 您使用此发送一个原始的HTTP请求。通常，如果你需要不常规的东西，你会需要这个方法。在大多数情况下，你应该更喜欢send_request_cgi。如果你想了解这个方法的工作原理，请查看Rex::Proto::Http::Client#request_raw的文档。
* send_request_cgi - 您使用它来发送更多CGI兼容的HTTP请求。如果你的请求包含查询字符串(或POST数据)，那么你应该使用这个。如果你想了解这种方法如何工作，请查看 Rex::Proto::Http::Client#request_cgi
这是一个`send_request_cgi`非常基本的例子
~~~
	send_request_cgi({
		'method'   => 'GET',
		'uri'      => '/hello_world.php',
		'vars_get' => {
			'param_1' => 'abc',
			'param_2' => '123'
		}
	})
~~~
请注意：send_request_raw和send_request_cgi如果超时会返回一个nil，请在处理返回值的时候考虑这个情况

### URI解析
在发送HTTP请求之前，您很可能必须做一些URI解析。这是一个棘手的任务，因为有时当你加入路径，你可能会不小心得到双斜线，如：“/test//index.php”。或者由于某种原因，你又缺少一个斜线。这些确实是常犯的错误。所以这里是你如何安全地处理它
1 - 将您的默认URI数据存储选项注册为“TARGETURI”：
~~~
	register_options(
		[
			OptString.new('TARGETURI', [true, 'The base path to XXX application', '/xxx_v1/'])
		], self.class)
~~~
2-  加载你的TARGETURI target_uri，这样的URI输入将会被验证，然后你得到一个真实的URI对象
在这个例子中，我们将只加载路径：
~~~
uri = target_uri.path
~~~
3- 当您想要加入另一个URI时，请始终使用[`normalize_uri`](https://rapid7.github.io/metasploit-framework/api/Msf/Exploit/Remote/HttpClient.html#normalize_uri-instance_method):
例子
~~~
# Returns: "/xxx_v1/admin/upload.php"
	uri = normalize_uri(uri, 'admin', 'upload.php')
~~~

4 - 当您完成URI的规范化后，即可使用send_request_cgi或send_request_raw

请注意：该normalize_uri方法将始终遵循这些规则：

该URI应该始终以斜线开头。
你将不得不决定是否需要结尾的斜杠。
不应该有双斜杠。

### 一个完整例子
~~~

	require 'msf/core'

	class MetasploitModule < Msf::Auxiliary

		include Msf::Exploit::Remote::HttpClient

		def initialize(info = {})
			super(update_info(info,
				'Name'           => 'HttpClient Example',
				'Description'    => %q{
					Do a send_request_cgi()
				},
				'Author'         => [ 'sinn3r' ],
				'License'        => MSF_LICENSE
			))

			register_options(
				[
					OptString.new('TARGETURI', [true, 'The base path', '/'])
				], self.class)
		end


		def run
			uri = target_uri.path

			res = send_request_cgi({
				'method'   => 'GET',
				'uri'      => normalize_uri(uri, 'admin', 'index.phpp'),
				'vars_get' => {
					'p1' => "This is param 1",
					'p2' => "This is param 2"
				}
			})

			if res && res.code == 200
				print_good("I got a 200, awesome")
			else
				print_error("No 200, feeling blue")
			end
		end
	end
~~~

### 使用Burp套件
Burp Suite是一个有用的工具，用于在使用HttpClient开发模块的同时检查或修改HTTPS流量。尝试这个：
1. 启动burp `java -jar burpsuite.jar`
2. 在Burp中，单击“proxy”选项卡，然后单击“option”。在那里配置代理侦听器。在这个例子中，假设我们在6666端口上有一个监听器。
3. 一旦Burp侦听器启动，启动msfconsole并加载您正在处理的模块
4. 输入： set Proxies HTTP:127.0.0.1:6666
5. 继续运行模块，Burp应拦截HTTPS流量
请注意，Burp仅支持HttpClient的HTTPS。这个问题只针对Burp和Metasploit。

如果您需要检查HttpClient的HTTP通信，解决方法是在模块中添加以下方法。这将覆盖HttpClient的send_request_ *方法，并返回修改的输出：
~~~

~~~
你也可以为send_request_raw做同样的事情。

### 其他常见问题
1 -我可以一起使用vars_get和vars_post么
是。当你提供一个散列vars_get,基本上它意味着“把所有的数据在查询字符串”.当你提供一个散列vars_post，意味着“把所有这些数据放在正文中”。他们都将发送相同的请求。当然，你确实需要确保你在使用send_request_cgi。

2 - 我由于一些奇怪的原因不能使用vars_get或vars_post，该怎么办？
在代码中提及这个问题(作为注释)。如果你不能使用vars_post，你可以尝试一下data键，它会发送你的原始数据。通常情况下，解决问题的最常见的解决方案vars_get是将您的东西留在uri关键位置。msftidy会标记这个，但只是作为“Info”而不是警告，这意味着你仍然应该通过msftidy。如果这是一个常见问题，我们可以随时更改msftidy。

3 - 我需要手动进行基本身份验证吗？
您不需要在请求中手动执行基本身份验证，因为HttpClient应该自动为您执行此操作。您只需在数据存储区选项中设置用户名和密码，然后当Web服务器询问时，mixin将使用该用户名和密码。


