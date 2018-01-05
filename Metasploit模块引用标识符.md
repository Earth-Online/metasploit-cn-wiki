## Metasploit模块引用标识符
Metasploit模块中的引用是与模块相关的信息的来源。这可以链接到漏洞咨询，新闻文章，关于模块使用的特定技术的博客文章，特定推文等。越多越好。但是，您不应该将其用作广告形式。

### 支持的参考标识符列表

ID  | Source | Code Example
------------- | ------------- | -------------
CVE  | cvedetails.com | ```['CVE', '2014-9999']```
CWE | cwe.mitre.org | ```['CWE', '90']```
BID | securityfocus.com | ```['BID', '1234']```
MSB | technet.microsoft.com | ```['MSB', 'MS13-055']```
EDB | exploit-db.com | ```['EDB', '1337']```
US-CERT-VU | kb.cert.org | ```['US-CERT-VU', '800113']```
ZDI | zerodayinitiative.com | ```['ZDI', '10-123']```
WPVDB | wpvulndb.com | ```['WPVDB', '7615']```
PACKETSTORM | packetstormsecurity.com | ```['PACKETSTORM', '132721']```
URL | anything | ```['URL', 'http://example.com/blog.php?id=123']```
AKA | anything | ```['AKA', 'shellshock']```
### 在模块中引用的示例
```ruby
require 'msf/core'

class MetasploitModule < Msf::Exploit::Remote
  Rank = NormalRanking

  def initialize(info={})
    super(update_info(info,
      'Name'           => "Code Example",
      'Description'    => %q{
        This is an example of a module using references
      },
      'License'        => MSF_LICENSE,
      'Author'         => [ 'Unknown' ],
      'References'     =>
        [
          [ 'CVE', '2014-9999' ],
          ['BID', '1234'],
          ['URL', 'http://example.com/blog.php?id=123']
        ],
      'Platform'       => 'win',
      'Targets'        =>
        [
          [ 'Example', { 'Ret' => 0x41414141 } ]
        ],
      'Payload'        =>
        {
          'BadChars' => "\x00"
        },
      'Privileged'     => false,
      'DisclosureDate' => "Apr 1 2014",
      'DefaultTarget'  => 0))
  end

  def exploit
    print_debug('Hello, world')
  end

end
```

