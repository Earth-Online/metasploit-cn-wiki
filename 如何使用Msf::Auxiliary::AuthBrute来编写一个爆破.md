这个```Msf::Auxiliary::AuthBrute``` mixin使用来编程一个login模块应该不是很长,你应该尝试我们的[LoginScanner API](https://github.com/rapid7/metasploit-framework/wiki/Creating-Metasploit-Framework-LoginScanners)代替.不过仍然需要一些数据存储选项，所以让我们快速浏览它们。 
### Regular options

* **USERNAME** - (String) 一个用于登录的特定用户名
* **PASSWORD** - (String) 一个用于登录的特定密码
* **USER_FILE** - (String)包含用户名的文件,每行一个。 
* **PASS_FILE** - (String) 包含密码的文件，每行一个。
* **USERPASS_FILE** - (String)包含用空格分隔的用户和密码的文件，每行一对。 
* **BRUTEFORCE_SPEED** - (Integer)爆破多快 从 0 到 5.
* **VERBOSE** - (Boolean)是否为所有尝试打印输出 
* **BLANK_PASSWORDS** - (Boolean) 为所有用户名尝试空密码
* **USER_AS_PASS** - (Boolean) 为所有用户名尝试相同密码
* **DB_ALL_CREDS** - (Boolean)尝试存储在当前数据库中的每个用户/密码对。 
* **DB_ALL_USERS** - (Boolean) 将当前数据库中的所有用户添加到列表中
* **STOP_ON_SUCCESS** - (Boolean) 成功后停止

### Advanced options

* **REMOVE_USER_FILE** - (Boolean) 在模块完成后自动删除 USER_FILE
* **REMOVE_PASS_FILE** - (Boolean) 在模块完成后自动删除 PASS_FILE
* **REMOVE_USERPASS_FILE** - (Boolean) 在模块完成后自动删除 USERPASS_FILE
* **MaxGuessesPerService** - (Integer) 每个服务实例尝试登录的最大数字.如果为0或非数字,则不使用 
* **MaxMinutesPerService** - (Integer) 爆破服务实例的最大时间(分钟).如果为0或数字,则不使用
* **MaxGuessesPerUser** - (Integer)对服务实例的特定用户名的最大爆破数量. 请注意，用户在不同的服务中被认为是唯一的，所以10.1.1.1:22的用户和10.2.2.2:22的用户是不同的，两者都将被尝试达到MaxGuessesPerUser限制。 如果为零或非数字，则不使用

### 参考

https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/auxiliary/auth_brute.rb
