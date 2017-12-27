##  创建Metasploit框架　LoginScanners模块
那么，你想在Metasploit中创建一个Login Scanner模块，呃？在开始之前，你需要知道几件事情。本文将试图说明创建一个有效的暴力/登录扫描器模块所涉及的所有部分。

[TOC]

### 凭据对象
Metasploit::Framework::Credential (lib/metasploit/framework/credential.rb)
这些对象代表了我们现在如何思考凭证的最基本概念。

* Public：证书的公共部分是指可以公开的部分。在几乎所有情况下，这是用户名。
* Private：证书的私人部分，这是应该是一个秘密的部分。这目前代表：密码，SSH密钥，NTLM哈希等
* Private Type：这个定义了上面定义了什么类型的私人证件
* Realm：这表示证书有效的认证区域。这是身份验证过程的一个重要部分。例子包括：活动目录域，Postgres数据库等
* Realm　key：这定义了领域属性代表什么类型的领域
* Paired：此属性是一个布尔值，用于设置凭据是否必须同时具有公有和私有两种属性
所有LoginScanner都使用Credential对象作为其尝试登录的基础。

### Result Objects

Metasploit::Framework::LoginScanner::Result (lib/metasploit/framework/login_scanner/result.rb)

这些是由扫描产生的对象！ 方法在每个LoginScanner上。 他们包含：

* Access Level: 可选的.访问级别，可以描述登录尝试授予的访问级别.
* Credential : 实现这结果的Credential 对象
* spoof: 一个可选的证明字符串。显示为什么我们认为结果是有效的
* Status: 登录尝试的状态。 这些值来自于
` Metasploit::model::Login::Status`
示例 "Incorrect", "Unable to Connect", "Untried" 

### CredentialCollection
Metasploit::Framework::CredentialCollection（lib/metasploit/ramework /credential_collection.rb）

这个类用于从模块获取数据存储选项和从每个方法中生成Credential对象。它需要wordlist文件，以及直接的用户名和密码选项。它也需要是否尝试将用户名作为密码和空白密码选项。
它可以作为LoginScanner上的cred_details传入，并响应#each并生成已制作的凭证。

示例(modules/auxiliary/scanner/ftp/ftp_login.rb):
~~~
cred_collection = Metasploit::Framework::CredentialCollection.new(
        blank_passwords: datastore['BLANK_PASSWORDS'],
        pass_file: datastore['PASS_FILE'],
        password: datastore['PASSWORD'],
        user_file: datastore['USER_FILE'],
        userpass_file: datastore['USERPASS_FILE'],
        username: datastore['USERNAME'],
        user_as_pass: datastore['USER_AS_PASS'],
        prepended_creds: anonymous_creds
    )

~~~

### LoginScanner 基础
Metasploit::Framework::LoginScanner::Base (lib/metasploit/framework/login_scanner/base.rb)
这是一个Ruby模块，包含所有LoginScanners的所有基本行为。所有的LoginScanner类都应该包含这个模块。
此行为的规范保存在共享示例组中。 您的LoginScanner规范应使用以下语法来包含这些测试：
~~~
it_behaves_like 'Metasploit::Framework::LoginScanner::Base', has_realm_key: false, has_default_realm: false
~~~
其中has_realm_key和has_default_realm应该根据您的LoginScanner是否有来设置。 
LoginScanner总是收集Crednetials来尝试登录一个主机和端口。 所以每个LoginScanner对象都只会尝试登录到一个特定的服务。

#### 属性
* connection_timeout: 一个连接等待多长时间超时
* cred_details: 一个在each中生成credentials的对象 (像一个 credentialCollection 或者一个数组)
* host: 目标主机地址
* port: 目标服务端口号
* proxies: 在连接中使用的任何代理（一些扫描可能不支持这个）
* stop_on_success: 是否在尝试成功登录后停止

#### 方法
* each_credential :你不用担心这个方法，请注意它在那里。 它遍历cred_details中的任何内容，进行一些规范化操作，并尝试确保每个Credential被正确设置以供给定的LoginScanner使用。 它在一个块中产生每个凭证。
~~~
def each_credential
           cred_details.each do |raw_cred|
             # This could be a Credential object, or a Credential Core, or an Attempt object
             # so make sure that whatever it is, we end up with a Credential.
             credential = raw_cred.to_credential

             if credential.realm.present? && self.class::REALM_KEY.present?
               credential.realm_key = self.class::REALM_KEY
               yield credential
             elsif credential.realm.blank? && self.class::REALM_KEY.present? && self.class::DEFAULT_REALM.present?
               credential.realm_key = self.class::REALM_KEY
               credential.realm     = self.class::DEFAULT_REALM
               yield credential
             elsif credential.realm.present? && self.class::REALM_KEY.blank?
               second_cred = credential.dup
               # Strip the realm off here, as we don't want it
               credential.realm = nil
               credential.realm_key = nil
               yield credential
               # Some services can take a domain in the username like this even though
               # they do not explicitly take a domain as part of the protocol.
               second_cred.public = "#{second_cred.realm}\\#{second_cred.public}"
               second_cred.realm = nil
               second_cred.realm_key = nil
               yield second_cred
             else
               yield credential
             end
           end
         end
~~~

* set_sane_defaults: 这个方法将被每个特定的Loginscanner覆盖。 这是在初始化程序的结束调用的，并为它们具有并且在初始化程序中没有给定具体的值的属性设置理想的默认值，
~~~
# This is a placeholder method. Each LoginScanner class
         # will override this with any sane defaults specific to
         # its own behaviour.
         # @abstract
         # @return [void]
         def set_sane_defaults
           self.connection_timeout = 30 if self.connection_timeout.nil?
         end
~~~

* attempt_login:这个方法只是Base mixin中的一个存根。 在每个LoginScanner类中将覆盖，以包含采用一个Credential对象的逻辑，并使用它来针对目标服务进行登录尝试。它返回一个::Metasploit::Framework::LoginScanner::Result 对象， 包含有关该尝试结果的所有信息。 举一个例子，让我们看一下来自Metasploit::Framework::LoginScanner::FTP (lib/metasploit/framework/login_scanner/ftp.rb)的attempt_login方法
~~~
# (see Base#attempt_login)
       def attempt_login(credential)
         result_options = {
             credential: credential
         }

         begin
           success = connect_login(credential.public, credential.private)
         rescue ::EOFError,  Rex::AddressInUse, Rex::ConnectionError, Rex::ConnectionTimeout, ::Timeout::Error
           result_options[:status] = Metasploit::Model::Login::Status::UNABLE_TO_CONNECT
           success = false
         end


         if success
           result_options[:status] = Metasploit::Model::Login::Status::SUCCESSFUL
         elsif !(result_options.has_key? :status)
           result_options[:status] = Metasploit::Model::Login::Status::INCORRECT
         end

         ::Metasploit::Framework::LoginScanner::Result.new(result_options)

       end
~~~
* scan! :这个方法是你将要关心的主要方法。 这个方法做了几件事。
1. 它调用有效！ 它将检查类的所有验证，如果发生任何失败。并抛出一个Metasploit::Framework::LoginScanner::Invalid。此错误将包含任何失败验证的全部错误消息
2. 它跟踪连接错误计数，并救援。如果我们有太多的连接错误或连续太多连接错误
3. 它通过在一个块调用each_credential来使用所有凭证
4. 在这个块中，它将每个凭证传递给#attempt_login
5. 它会生成Result对象放入传递的块中
6. 如果设置了stop_on_success，如果结果是成功的，它也会提前退出

~~~

# Attempt to login with every {Credential credential} in
          # {#cred_details}, by calling {#attempt_login} once for each.
          #
          # If a successful login is found for a user, no more attempts
          # will be made for that user.
          #
          # @yieldparam result [Result] The {Result} object for each attempt
          # @yieldreturn [void]
          # @return [void]
          def scan!
            valid!

            # Keep track of connection errors.
            # If we encounter too many, we will stop.
            consecutive_error_count = 0
            total_error_count = 0

            successful_users = Set.new

            each_credential do |credential|
              next if successful_users.include?(credential.public)

              result = attempt_login(credential)
              result.freeze

              yield result if block_given?

              if result.success?
                consecutive_error_count = 0
                break if stop_on_success
                successful_users << credential.public
              else
                if result.status == Metasploit::Model::Login::Status::UNABLE_TO_CONNECT
                  consecutive_error_count += 1
                  total_error_count += 1
                  break if consecutive_error_count >= 3
                  break if total_error_count >= 10
                end
              end
            end
            nil
          end
~~~
#### 常量
虽然没有在Base上定义，但是每个LoginScanner都有一系列可以在其上面定义的常量来帮助处理关键行为。
* DEFAULT_PORT:DEFAULT_PORT是一个与set_sane_defaults一起使用的简单常量。如果端口没有被用户设置，它将使用DEFAULT_PORT。这被放在一个常量，所以它可以从扫描仪外部快速引用。
LoginScanner命名空间方法classes_for_services使用这两个常量。这个被调用的方法Metasploit::Framework::LoginScanner.classes_for_service(<Mdm::service>)实际上会返回一个LoginScanner类的数组，这对于针对这个特定的Service可能是有用的。

* LIKELY_PORTS：这个常量保存了n个端口号，这对于使用这个扫描器来说可能是有用的。
* LIKELY_SERVICE_NAMES：如上所述，服务名称的字符串而不是端口号。
* PRIVATE_TYPES：这包含表示它所支持的不同私有证书类型的符号数组。它应该总是匹配私人类的demodulize结果，即password ntlm_hash ssh_key

这些常量是必须处理域名(如AD域或数据库名称)的LoginScanners
* REALM_KEY: 这个扫描器希望处理的领域的类型.应始终是来自metasploit::Model::Login::Status的常量

* DEFAULT_REALM:一些扫描仪有一个默认的领域(比如AD域名的WORKSTATION).如果将凭证给需要领域的扫描器，但凭证没有领域，则将该值作为领域的值添加到证书中。

* CAN_GET_SESSION: 这应该是TURE或者FALES的，我们是否希望我们能够以某种方式获得从这个扫描仪发现的凭证会话。

示例１( Metasploit::Framework::LoginScanner::FTP)
~~~
DEFAULT_PORT         = 21
       LIKELY_PORTS         = [ DEFAULT_PORT, 2121 ]
       LIKELY_SERVICE_NAMES = [ 'ftp' ]
       PRIVATE_TYPES        = [ :password ]
       REALM_KEY           = nil
~~~

示例2( Metasploit::Framework::LoginScanner::SMB)
~~~
CAN_GET_SESSION      = true
        DEFAULT_REALM        = 'WORKSTATION'
        LIKELY_PORTS         = [ 139, 445 ]
        LIKELY_SERVICE_NAMES = [ "smb" ]
        PRIVATE_TYPES        = [ :password, :ntlm_hash ]
        REALM_KEY            = Metasploit::Model::Realm::Key::ACTIVE_DIRECTORY_DOMAIN

~~~

### 把它们放在一个模块中

所以，现在你希望有从所有移动部分创建一个LoginScanner的好想法。下一步是在实际模块中使用全新的LoginScanner。

我们来看看ftp_login模块：
`def run_host(ip)`

每个Bruteforce/Login模块都应该是一个扫描器，并且应该使用每个RHOST调用一次的run_host方法。

#### 凭证收集
~~~
 cred_collection = Metasploit::Framework::CredentialCollection.new(
        blank_passwords: datastore['BLANK_PASSWORDS'],
        pass_file: datastore['PASS_FILE'],
        password: datastore['PASSWORD'],
        user_file: datastore['USER_FILE'],
        userpass_file: datastore['USERPASS_FILE'],
        username: datastore['USERNAME'],
        user_as_pass: datastore['USER_AS_PASS'],
        prepended_creds: anonymous_creds
    )
~~~
所以在这里我们看到使用数据存储选项创建CredentialCollection。 我们传递了凭证创建的选项，例如密码表，原始用户名和密码，是否尝试将用户名作为密码，以及是否尝试空白密码。
你也会注意到这里有个选项prepended_creds。 FTP只是是使用这个的模块之一，但它通常通过CredentialCollection可以使用。这个选项是一个Metasploit::Framework::Credential 的数组。在使用其他凭证对象前会被使用。FTP使用这个来处理匿名FTP访问的测试。

#### 初始化扫描
~~~
scanner = Metasploit::Framework::LoginScanner::FTP.new(
        host: ip,
        port: rport,
        proxies: datastore['PROXIES'],
        cred_details: cred_collection,
        stop_on_success: datastore['STOP_ON_SUCCESS'],
        connection_timeout: 30
    )
~~~
这里我们实际上创建了我们的Scanner对象。 我们根据模块已知的数据设置IP和端口。 我们可以从数据存储中提取任何用户提供的代理数据。 我们也从数据存储中取出stop_on_success。 信用详情对象由我们的cred_collection填充，这将无形地处理我们所有的凭证生成。

这将给我们一个scaner对象　一切准备好了。让我们开始

#### 扫描代码块
~~~
scanner.scan! do |result|
      credential_data = result.to_h
      credential_data.merge!(
          module_fullname: self.fullname,
          workspace_id: myworkspace_id
      )
      if result.success?
        credential_core = create_credential(credential_data)
        credential_data[:core] = credential_core
        create_credential_login(credential_data)

        print_good "#{ip}:#{rport} - LOGIN SUCCESSFUL: #{result.credential}"
      else
        invalidate_login(credential_data)
        print_status "#{ip}:#{rport} - LOGIN FAILED: #{result.credential} (#{result.status}: #{result.proof})"
      end
    end
~~~
这是这件事的真正核心。我们调用扫描！
在我们的扫描仪，并通过一个代码块。正如我们之前提到的那样，扫描器将每个尝试的Result对象放到该块中。我们检查结果的状态，看看它是否成功。
result对象现在作为一个.to_h方法，它返回一个与我们的凭证创建方法兼容的哈希。我们把这个哈希合并到我们模块的特定信息和工作区ID中。
在成功的情况下，我们建立一些信息散列并调用create_credential。

这是在metasploit-credential gem下lib / metasploit / credential / creation.rb中找到的一种方法调用Metasploit::Credential::Creation

mixin包含在Report mixin中，所以如果你的模块包含了mixin，你可以免费获得这些方法。

create_credential创建一个Metasploit::Credential::Core。
然后，我们把这个核心，服务数据，并与一些额外的数据合并。
 这些附加数据包括访问级别，当前时间(更新在Metasploit::Credential::Login的last_attempted_at)，状态。
 完成。对于一个成功　我们把结果输出到控制台
 对于错误的情况，我们调用invalidate_login 方法。这个方法也是来自Creation mixin
 这个方法查看credential:service pair.是否有一个login对象存在这个。如果是，我们将它的状态更新到我们从scaner返回得到的状态。
这主要是为了具有未尝试状态的Post模块创建的Login对象。

### ftp_login最终图
综合起来，我们得到一个新的ftp_login模块，看起来像这样：
~~~
##
# This module requires Metasploit: http//metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

require 'msf/core'
require 'metasploit/framework/credential_collection'
require 'metasploit/framework/login_scanner/ftp'

class Metasploit3 < Msf::Auxiliary

  include Msf::Exploit::Remote::Ftp
  include Msf::Auxiliary::Scanner
  include Msf::Auxiliary::Report
  include Msf::Auxiliary::AuthBrute

  def proto
    'ftp'
  end

  def initialize
    super(
      'Name'        => 'FTP Authentication Scanner',
      'Description' => %q{
        This module will test FTP logins on a range of machines and
        report successful logins.  If you have loaded a database plugin
        and connected to a database this module will record successful
        logins and hosts so you can track your access.
      },'Author'      => 'todb',
      'References'     =>
        [
          [ 'CVE', '1999-0502'] # Weak password
        ],
      'License'     => MSF_LICENSE
    )

    register_options(
      [
        Opt::RPORT(21),
        OptBool.new('RECORD_GUEST', [ false, "Record anonymous/guest logins to the database", false])
      ], self.class)

    register_advanced_options(
      [
        OptBool.new('SINGLE_SESSION', [ false, 'Disconnect after every login attempt', false])
      ]
    )

    deregister_options('FTPUSER','FTPPASS') # Can use these, but should use 'username' and 'password'
    @accepts_all_logins = {} 
    end


  def run_host(ip)
    print_status("#{ip}:#{rport} - Starting FTP login sweep")

    cred_collection = Metasploit::Framework::CredentialCollection.new(
        blank_passwords: datastore['BLANK_PASSWORDS'],
        pass_file: datastore['PASS_FILE'],
        password: datastore['PASSWORD'],
        user_file: datastore['USER_FILE'],
        userpass_file: datastore['USERPASS_FILE'],
        username: datastore['USERNAME'],
        user_as_pass: datastore['USER_AS_PASS'],
        prepended_creds: anonymous_creds
    )

    scanner = Metasploit::Framework::LoginScanner::FTP.new(
        host: ip,
        port: rport,
        proxies: datastore['PROXIES'],
        cred_details: cred_collection,
        stop_on_success: datastore['STOP_ON_SUCCESS'],
        connection_timeout: 30
    )

    scanner.scan! do |result|
      credential_data = result.to_h
      credential_data.merge!(
          module_fullname: self.fullname,
          workspace_id: myworkspace_id
      )
      if result.success?
        credential_core = create_credential(credential_data)
        credential_data[:core] = credential_core
        create_credential_login(credential_data)

        print_good "#{ip}:#{rport} - LOGIN SUCCESSFUL: #{result.credential}"
      else
        invalidate_login(credential_data)
        print_status "#{ip}:#{rport} - LOGIN FAILED: #{result.credential} (#{result.status}: #{result.proof})"
      end
    end

  end
  # Always check for anonymous access by pretending to be a browser.
  def anonymous_creds
    anon_creds = [ ]
    if datastore['RECORD_GUEST']
      ['IEUser@', 'User@', 'mozilla@example.com', 'chrome@example.com' ].each do |password|
        anon_creds << Metasploit::Framework::Credential.new(public: 'anonymous', private: password)
      end
    end
    anon_creds
  end

  def test_ftp_access(user,scanner)
    dir = Rex::Text.rand_text_alpha(8)
    write_check = scanner.send_cmd(['MKD', dir], true)
    if write_check and write_check =~ /^2/
      scanner.send_cmd(['RMD',dir], true)
      print_status("#{rhost}:#{rport} - User '#{user}' has READ/WRITE access")
      return 'Read/Write'
    else
      print_status("#{rhost}:#{rport} - User '#{user}' has READ access")
      return 'Read-only'
    end
  end


end

~~~