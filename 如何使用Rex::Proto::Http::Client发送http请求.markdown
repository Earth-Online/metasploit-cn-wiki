**注意: 这个文档可能需要审查.**
这个REX库(ruby 扩展库)是Metasploit Framework架构的最基本的部分。模块通常不直接与Rex直接交互,相反他们依赖与框架核心和它的mixin来实现更好的代码分享.如果您是Metasploit模块开发人员the [lib/msf/core](https://github.com/rapid7/metasploit-framework/tree/master/lib/msf/core) 目录应该能满足你绝大部分的需要.如果您正在编写一个使用HTTP的模块,那么[Msf::Exploit::Remote::HttpClient](https://github.com/rapid7/metasploit-framework/wiki/How-to-Send-an-HTTP-Request-Using-HTTPClient) mixin(可以在[lib/msf/core/exploit/http/client](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/http/client.rb)找到)很可能是你想要的
然而,在一些情况下,你实际不能使用HttpClient mixin.最常见的是使用[LoginScanner API](https://github.com/rapid7/metasploit-framework/wiki/Creating-Metasploit-Framework-LoginScanners)编写基于表单的登录模块.如果发现你在这种情况,使用 [Rex::Proto::Http::Client](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/proto/http/client.rb).
### 初始化Rex::Proto::Http::Client
Rex::Proto::Http::Client初始化程序创建一个新的http客户端实例,最重要的是
```ruby
def initialize(host, port = 80, context = {}, ssl = nil, ssl_version = nil, proxies = nil, username = '', password = '')
```
正如你可以使用的,只是需要一个host参数,rest是可选的.让我们快速的回顾一下

| Argument name | Data type | Description |
| ------------- | --------- | ----------- |
| host | String | Target host IP |
| port | Fixnum | Target host port |
| context | Hash | Determines what is responsible for requesting that a socket can be created |
| ssl | Boolean | True to enable it |
| ssl_version | String | SSL2, SSL3, or TLS1 |
| proxies | String | Configure a proxy |
| username | String | Username for automatic authentication |
| password | String | Password for automatic authentication |

初始化Rex::Proto::Http::Client的代码示例
```ruby
cli = Rex::Proto::Http::Client.new(rhost, rport, {}, true, 8181, proxies, 'username', 'password')
```

## 制造一个http请求
即使我们的这个文档的主要主题是关于Rex::Proto::Http::Client,它其实不知道怎么制造http请求.相反[Rex::Proto::Http::ClientRequest](https://github.com/rapid7/metasploit-framework/blob/master/lib/rex/proto/http/client_request.rb)实际上是所有metasploit的http请求的母亲.
那么Rex::Proto::Http::ClientRequest 如何产生一个http请求?你看到儿子,这一切都是从Rex::Proto::Http::Client要求一个#request_cgi或者#request_raw method开始.不同之处在于如果使用#request_cgi，请求的目的是CGI兼容，在大多数情况下，这是你想要的。如果使用#request_raw,技术上这意味着更少的选项,更少的CGI兼容。
原始http请求支持以下选项

| Option/key name | Data type | Description |
| --------------- | --------- | ----------- |
| query | String | Raw GET query string |
| data | String | Raw POST data string |
| uri | String | Raw URI string |
| ssl | Boolean | True to use https://, otherwise http:// |
| agent | String | User-Agent. Default is: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)|
| method | String | HTTP method |
| proto | String | Protocol |
| version | String | Version |
| vhost | String | Host header |
| port | Fixnum | Port for the host header |
| authorization | String | The authorization header |
| cookie | String | The cookie header |
| connection | String | The connection header |
| headers | Hash | A hash of custom headers. Safer than raw_headers |
| raw_headers | String | A string of raw headers |
| ctype | String | Content type |

使用#request_raw选项的一个例子
```ruby
# cli is a Rex::Proto::Http::Client object
req = cli.request_raw({
	'uri'    =>'/test.php',
	'method' => 'POST',
	'data'   => 'A=B'
})
```

**request_cgi继承了以上所有内容**，还有更多：

| Option/key name | Data type | Description |
| --------------- | --------- | ----------- |
| query | String | Raw GET query string |
| data | String | Raw POST data string |
| uri | String | Raw URI string |
| ssl | Boolean | True to use https://, otherwise http:// |
| agent | String | User-Agent. Default is: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)|
| method | String | HTTP method |
| proto | String | Protocol |
| version | String | Version |
| vhost | String | Host header |
| port | Fixnum | Port for the host header |
| authorization | String | The authorization header |
| cookie | String | The cookie header |
| connection | String | The connection header |
| headers | Hash | A hash of custom headers. Safer than raw_headers |
| raw_headers | String | A string of raw headers |

一个使用了一个#request_cgi选项的例子
```ruby
# cli is a Rex::Proto::Http::Client object
req = cli.request_cgi({
	'uri'      =>'/test.php',
	'vars_get' => {
		'param1' => 'value',
		'param2' => 'value'
	}
})
```

## 发送一个http请求
这是一个如何使用#request_cgi 或 #request_raw实际上访问一个http系统的例子
**#request_cgi**

```ruby
cli = Rex::Proto::Http::Client.new(rhost),
cli.connect
req = cli.request_cgi({'uri'=>'/'})
res = cli.send_recv(req)
cli.close
```

**#request_raw**

```ruby
cli = Rex::Proto::Http::Client.new(rhost),
cli.connect
req = cli.request_raw({'uri'=>'/'})
res = cli.send_recv(req)
cli.close
```

## 配置高级选项
**回避选项** 
Rex::Proto::Http::Client也有它自己的回避选项.当你使用Rex::Proto::Http::ClientRequest 制造一个http请求时你能设置它们.或者你也能在一个#set_config方法设置.主要区别是如果你使用#set_config,你应该使得那些选项用户可以配置.
| Option | Data type | Default | Known configurable option |
| ------ | --------- | ------- | ------------- |
| encode_params | Boolean | true | N/A |
| encode | Boolean | false | N/A |
| uri_encode_mode | String | hex-normal | HTTP::uri_encode_mode |
| uri_encode_count | Fixnum | 1 | N/A |
| uri_full_url | Boolean | false | HTTP::uri_full_url |
| pad_method_uri_count | Fixnum | 1 | HTTP::pad_method_uri_count |
| pad_uri_version_count | Fixnum | 1 | HTTP::pad_uri_version_count |
| pad_method_uri_type | String | space | HTTP::pad_method_uri_type |
| pad_uri_version_type | String | space | HTTP::pad_uri_version_type |
| method_random_valid | Boolean | false | HTTP::method_random_valid |
| method_random_invalid | Boolean | false | HTTP::method_random_invalid |
| method_random_case | Boolean | false | HTTP::method_random_case |
| version_random_valid | Boolean | false | N/A |
| version_random_invalid| Boolean | false | N/A |
| version_random_case | Boolean | false | N/A |
| uri_dir_self_reference | Boolean | false | HTTP::uri_dir_self_reference |
| uri_dir_fake_relative | Boolean | false | HTTP::uri_dir_fake_relative |
| uri_use_backslashes | Boolean | false | HTTP::uri_use_backslashes |
| pad_fake_headers | Boolean | pad_fake_headers| HTTP::pad_fake_headers |
| pad_fake_headers_count | Fixnum | 16 | HTTP::pad_fake_headers_count |
| pad_get_params | Boolean | false | HTTP::pad_get_params |
| pad_get_params_count | Boolean | 8 | HTTP::pad_get_params_count |
| pad_post_params | Boolean | false | HTTP::pad_post_params |
| pad_post_params_count | Fixnum | 8 | HTTP::pad_post_params_count |
| uri_fake_end | Boolean | false | HTTP::uri_fake_end |
| uri_fake_params_start | Boolean | false | HTTP::uri_fake_params_start |
| header_folding | Boolean | false | HTTP::header_folding |
| chunked_size | Fixnum | 0 | N/A |

**NTLM Options**

HTTP 身份验证 在  Rex::Proto::Http::Client是自动的,当涉及到NTLM提供程序时，它将获得自己的选项。您必须使用#set_config方法来设置它们

| Option | Data type | Default | Known configurable option |
| ------ | --------- | ------- | ------------- |
| usentlm2_session | Boolean | true | NTLM::UseNTLM2_session |
| use_ntlmv2 | Boolean | true | NTLM::UseNTLMv2 |
| send_lm | Boolean | true | NTLM::SendLM |
| send_ntlm | Boolean | true | NTLM::SendNTLM |
| SendSPN | Boolean | true | NTLM::SendSPN |
| UseLMKey | Boolean | false | NTLM::UseLMKey |
| domain | String | WORKSTATION | DOMAIN |
| DigestAuthIIS | Boolean | true | DigestAuthIIS |

**注意** “已知的配置选项”表示HttpClient有一个数据存储选项。如果你不能使用HttpClient，那么你将不得不考虑自己注册它们。

## url解析
Rex::Proto::Http::Client实际上不支持url解析,所以对于URI格式验证和格式化,你得自己做.你应该这样做
对于URI格式验证，我们推荐使用Ruby的URI模块。您可以使用HttpClient的#target_uri方法为例。
对于URI规则化，我们建议HttpClient的＃normalize_uri方法。

## 完整例子
```ruby
cli = Rex::Proto::Http::Client.new(rhost, rport, {}, ssl, ssl_version, proxies, user, pass)
cli.set_config(
  'vhost' => vhost,
  'agent' => datastore['UserAgent'],
  'uri_encode_mode'        => datastore['HTTP::uri_encode_mode'],
  'uri_full_url'           => datastore['HTTP::uri_full_url'],
  'pad_method_uri_count'   => datastore['HTTP::pad_method_uri_count'],
  'pad_uri_version_count'  => datastore['HTTP::pad_uri_version_count'],
  'pad_method_uri_type'    => datastore['HTTP::pad_method_uri_type'],
  'pad_uri_version_type'   => datastore['HTTP::pad_uri_version_type'],
  'method_random_valid'    => datastore['HTTP::method_random_valid'],
  'method_random_invalid'  => datastore['HTTP::method_random_invalid'],
  'method_random_case'     => datastore['HTTP::method_random_case'],
  'uri_dir_self_reference' => datastore['HTTP::uri_dir_self_reference'],
  'uri_dir_fake_relative'  => datastore['HTTP::uri_dir_fake_relative'],
  'uri_use_backslashes'    => datastore['HTTP::uri_use_backslashes'],
  'pad_fake_headers'       => datastore['HTTP::pad_fake_headers'],
  'pad_fake_headers_count' => datastore['HTTP::pad_fake_headers_count'],
  'pad_get_params'         => datastore['HTTP::pad_get_params'],
  'pad_get_params_count'   => datastore['HTTP::pad_get_params_count'],
  'pad_post_params'        => datastore['HTTP::pad_post_params'],
  'pad_post_params_count'  => datastore['HTTP::pad_post_params_count'],
  'uri_fake_end'           => datastore['HTTP::uri_fake_end'],
  'uri_fake_params_start'  => datastore['HTTP::uri_fake_params_start'],
  'header_folding'         => datastore['HTTP::header_folding'],
  'usentlm2_session'       => datastore['NTLM::UseNTLM2_session'],
  'use_ntlmv2'             => datastore['NTLM::UseNTLMv2'],
  'send_lm'                => datastore['NTLM::SendLM'],
  'send_ntlm'              => datastore['NTLM::SendNTLM'],
  'SendSPN'                => datastore['NTLM::SendSPN'],
  'UseLMKey'               => datastore['NTLM::UseLMKey'],
  'domain'                 => datastore['DOMAIN'],
  'DigestAuthIIS'          => datastore['DigestAuthIIS']
)
cli.connect
req = cli.request_cgi({'uri'=>'/'})
res = cli.send_recv(req)
cli.close
```


