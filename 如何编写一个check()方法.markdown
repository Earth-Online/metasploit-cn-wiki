在Metapsloit,exploit和辅助模块支持check命令 使得用户可以在开始使用模块之前确认漏洞的状态.这个功能是便利于那些需要在不弹出shell的情况下确认漏洞的人,并且可以用于快速识别网络上所有易受攻击或可能被利用的机器。
虽然漏洞确认不是metasploit的关注点,因为它不是像Nexpose这样的漏洞扫描器.我们通常鼓励人们实现check()方法来增加模块的价值.如果你写,一定要记住下面的条例

## check 方法输出
模块消息对用户来说是重要,因为它们通知它一直在做什么,和通常使得模块更好debug.但是,你也想要你的消息在详细模式,因为如果该检查针对多个目标使用，则会变得非常嘈杂。理想情况下，您只应使用这些打印方法：
| Method | Description |
| ------ | ----------- |
| **vprint_line()** | verbose version of print_line |
| **vprint_status()** | verbose version of print_status that begins with "[*]" |
| **vprint_error()** | verbose version of print_error that begins with "[x]" |
| **vprint_warning()** | verbose version of print_warning that begins with "[!]", in yellow |
| **vprint_debug()** | verbose versino of print_debug that begins with "[!]", in blue |

注意:如果目标存在漏洞,你不应该输出,因为你的方法返回一个确认码后框架会自动处理

## 确认码
只要你有一个确认漏洞状态,你应该返回一个确认码.确认码是定义在Msf::Exploit::CheckCode的常量,这些是你可以使用的
| Checkcode | Description |
| --------- | ----------- |
| **Exploit::CheckCode::Unknown** | Used if the module fails to retrieve enough information from the target machine, such as due to a timeout. |
| **Exploit::CheckCode::Safe** | Used if the check fails to trigger the vulnerability, or even detect the service. |
| **Exploit::CheckCode::Detected** | The target is running the service in question, but the check fails to determine whether the target is vulnerable or not. |
| **Exploit::CheckCode::Appears** | This is used if the vulnerability is determined based on passive reconnaissance. For example: version, banner grabbing, or simply having the resource that's known to be vulnearble. |
| **Exploit::CheckCode::Vulnerable** | Only used if the check is able to actually take advantage of the bug, and obtain some sort of hard evidence. For example: for a command execution type bug, get a command output from the target system. For a directory traversal, read a file from the target, etc. Since this level of check is pretty aggressive in nature, you should not try to DoS the host as a way to prove the vulnerability. |
| **Exploit::CheckCode::Unsupported** | The exploit does not support the check method. If this is the case, then you don't really have to add the check method. |

## 远程确认例子
这是一个如何编写Metasploit check的抽象例子
```ruby
#
# Returns a check code that indicates the vulnerable state on an app running on OS X
#
def check
  if exec_cmd_via_http("id") =~ /uid=\d+\(.+\)/
    # Found the correct ID output, good indicating our command executed
    return Exploit::CheckCode::Vulnerable
  end

  http_body = get_http_body
  if http_body
    if http_body =~ /Something CMS v1\.0/
      # We are able to find the version thefore more precise about the vuln state
      return Exploit::CheckCode::Appears
    elsif http_body =~ /Something CMS/
      # All we can tell the vulnerable app is running, but no more info to
      # determine the vuln
      return Exploit::CheckCode::Detected
    end
  else
    vprint_error("Unable to determine due to a HTTP connection timeout")
    return Exploit::CheckCode::Unknown
  end

  Exploit::CheckCode::Safe
end
```

注意: 如果你在编写一个使用```Msf::Auxiliary::Scanner``` mixin的辅助模块,你的方法声明应该像这样

```ruby
def check_host(ip)
  # Do your thing
end
```

### 本地exploit利用例子
大多数本地exploit check 是确认漏洞文件的版本,这被认为是被动的,因此他们应该标记Exploit::CheckCode::Appears.被动本地exploit check不代表他们是不可靠的,实际上,它们是没问题的.但是要符合Exploit::CheckCode::Vulnerable,你的check应该是额外的,这意味着要么以某种方式使程序返回易受攻击的响应，要么检查易受攻击的代码。
```ruby
def check
  check_str = Rex::Text.rand_text_alphanumeric(5)
  # ensure they are vulnerable to bash env variable bug
  if cmd_exec("env x='() { :;}; echo #{check_str}' bash -c echo").include?(check_str) &&
     cmd_exec("file '#{datastore['VMWARE_PATH']}'") !~ /cannot open/

     Exploit::CheckCode::Vulnerable
  else
    Exploit::CheckCode::Safe
  end
end
```

检查易受攻击的代码的一种方法是提供一个签名，看看它是否存在于易受攻击的进程中.以下是adobe_sandbox_adobecollabsync.rb的示例：

```ruby
# 'AdobeCollabSyncTriggerSignature' => "\x56\x68\xBC\x00\x00\x00\xE8\xF5\xFD\xFF\xFF"
# 'AdobeCollabSyncTrigger' => 0x18fa0

def check_trigger
  signature = session.railgun.memread(@addresses['AcroRd32.exe'] + target['AdobeCollabSyncTrigger'], target['AdobeCollabSyncTriggerSignature'].length)
  if signature == target['AdobeCollabSyncTriggerSignature']
    return true
  end

  return false
end

def check
  @addresses = {}
  acrord32 = session.railgun.kernel32.GetModuleHandleA("AcroRd32.exe")
  @addresses['AcroRd32.exe'] = acrord32["return"]
  if @addresses['AcroRd32.exe'] == 0
    return Msf::Exploit::CheckCode::Unknown
  elsif check_trigger
    return Msf::Exploit::CheckCode::Vulnerable
  else
    return Msf::Exploit::CheckCode::Detected
  end
end
```
另一个可能的检查方法是抓住易受攻击的文件,并使用Metasm.但是当然,这会慢很多,会产生更多的网络流量。