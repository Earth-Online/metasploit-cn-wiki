## 如何弃用Metasploit模块
Metasploit有一个非常具体的方式来弃用一个模块。为此，您必须使用Msf::Module::Deprecated mixin. 你必须使用这个mixin有两个原因
1. 您需要设置弃用日期。这样我们知道什么时候删除它，这是手动完成的
2. 您需要设置您希望弃用的模块的替代品

### 示例
使用 Msf::Module::Deprecated ，这是如何操作
1.  在您的模块`class metasploit3`下，包含以下内容
~~~
包括 Msf :: Module :: Deprecated
~~~
2.   a:使用`deprecated`方法分配弃用日期和替换模块
      ```deprecated(Date.new(2014, 9, 21), 'exploit/linux/http/dlink_upnp_exec_noauth')```
     b:可以使用定义DEPRECATION_DATE和DEPRECATION_REPLACEMENT常量替代
     ```
     DEPRECATION_DATE = Date.new(2014, 9, 21) # Sep 21
# The new module is exploit/linux/http/dlink_upnp_exec_noauth
DEPRECATION_REPLACEMENT = 'exploit/linux/http/dlink_upnp_exec_noauth'
     ```
 这样当用户载入模块时 他们应该就会看到这样的警告
 ```
 msf > use exploit/windows/misc/test 

[!] ************************************************************************
[!] *             The module windows/misc/test is deprecated!              *
[!] *              It will be removed on or about 2014-09-21               *
[!] *        Use exploit/linux/http/dlink_upnp_exec_noauth instead        *
[!] ************************************************************************
 ```
 
 
 ### 代码示例
 ```
 require 'msf/core'

class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Module::Deprecated

  deprecated(Date.new(2014, 9, 21), 'exploit/linux/http/dlink_upnp_exec_noauth')

  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'Msf::Module::Deprecated Example',
      'Description' => %q{
        This shows how to use Msf::Module::Deprecated.
      },
      'Author'      => [ 'sinn3r' ],
      'License'     => MSF_LICENSE,
      'References'  => [ [ 'URL', 'http://metasploit.com' ] ],
      'DisclosureDate' => 'Apr 01 2014',
      'Targets'        => [ [ 'Automatic', { } ] ],
      'DefaultTarget'  => 0
    ))
  end

  def exploit
    print_debug("Code example")
  end

end
 ```