当Metasploit Payloads `master`出现一个新的合并时，一个新的Ruby gem被构建并自动推送到RubyGems。这个新版本需要被合并到Metasploit Framework存储库中，以便包含这些更改。
为此，提交者必须：

* 在Metasploit Framework存储库中创建一个新的分支。
* 名字是有用的像`metasploit-payloads-<version>`
* 修改 `metasploit.gemspec`,以便为metasploit-payloads gem 指定新的版本号。
* 运行`bundle install`
* 从`data/meterpreter`中删除任何测试/开发二进制文件
* 运行`tools/modules/update_payload-cached_sizes.rb`
* 确保`Gemfile.lock`只包含与Metasploit Payload相关的更改。
* 按下列阶段进行`git`提交：
    * `Gemfile.lock`
    * `metasploit.gemspec`
    * 任何具有更新的payload的payload模块（通常这仅包括无阶段有效载荷）
* 提交暂存的文件。
* push分支给github。
* 创建合并请求。

完成

更新PR/提交例子可以在这里找到：https：//github.com/rapid7/metasploit-framework/pull/7666/files
