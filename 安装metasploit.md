## Metasploit Development Environment
这是一个关于怎么安装一个有效的`Metasploit`开发环境的指南
如果你只是想使用合法,切授权的`Metasploit`来进行黑客活动,我们建议你使用商业版`Metasploit`框架安装包或者这个开源安装包,这将处理你所有的依赖关系。
商业安装程序还包括升级到` Metasploit Pro` 的选项, 并一周两次更新,而开放源码的安装者将每晚更新

如果你是`kali linux` ` metasploit`是已经预先安装了的,查看`kali linux`如何开始使用`metasploit`并设置数据库

如果你想发展和贡献 `Metasploit`, 阅读这本指南应该让你在所有基于`debian`的系统使用
让我们开始吧

### 假设
1. 您有一个 `Debian-based` 的 Linux 环境
2. 您有一个不是`root`的用户。在本指南中, 我们使用的是`msfdev`。
3. 您有一个` GitHub` 帐户


### 下载开发依赖包
~~~
sudo apt-get -y install \
  autoconf \
  bison \
  build-essential \
  curl \
  git-core \
  libapr1 \
  libaprutil1 \
  libcurl4-openssl-dev \
  libgmp3-dev \
  libpcap-dev \
  libpq-dev \
  libreadline6-dev \
  libsqlite3-dev \
  libssl-dev \
  libsvn1 \
  libtool \
  libxml2 \
  libxml2-dev \
  libxslt-dev \
  libyaml-dev \
  locate \
  ncurses-dev \
  openssl \
  postgresql \
  postgresql-contrib \
  wget \
  xsel \
  zlib1g \
  zlib1g-dev
~~~
注意, 还没`Ruby` 不过我们很快就会得到。

### fork或clone metasploit
您可以按照github的`fork`指令操作, 但它基本上只是在`Metasploit`框架的页面的右上方,点击 `fork` 按钮

#### Clone
如果你有一个`fork`在 GitHub, 是时候把它拉到你的本地开发了。当然 您需要按照 GitHub 中的`clone`指令进行操作。
~~~
mkdir -p $HOME/git
cd $HOME/git
git clone git@github.com:YOUR_USERNAME_FOR_GITHUB/metasploit-framework
cd metasploit-framework
~~~

#### 设置git上游
首先, 如果你计划用最新的`Metasploitg-framework`Git仓库来更新你的本地克隆库, 你就会想要跟踪它。在您的`Metasploit-framework`库中, 进行以下操作:
~~~
git remote add upstream git@github.com:rapid7/metasploit-framework.git
git fetch upstream
git checkout -b upstream-master --track upstream/master
~~~

现在, 你有一个分支, 指向上游 (这个 `rapid7` fork), 这是不同于你自己的fork (原始的主分支, 指向`origin/master`)。你可能会发现有上游和master是不同的分支 (特别是如果你是一个 `Metasploi`t 提交, 因为这使得它不太可能不小心推到 `rapid7/master`)。

### 　下载rvm
大多数发行版不会与最新的 `Ruby` 一起使用一样的频率更新。因此, 我们将使用 `RVM`, 一个`Ruby` 版本管理器。你可以在这里读到指南: https://rvm.io/, 发现它是相当大。有些人喜欢` rbenv`。你可以使用`rbenv`, 但你要靠自己来确保你有一个正常的ruby版本。大多数的提交使用 `RVM`, 所以对于本指南, 我们要坚持下去。

首先, 您需要 RVM 分发的签名密钥:
~~~
curl -sSL https://rvm.io/mpapis.asc | gpg --import -
~~~

接下来, 获取 RVM 
~~~
curl -L https://get.rvm.io | bash -s stable
~~~

这是直接的传递到 bash, 这可能是一个敏感的问题。一种更长、更安全的方式:
~~~
curl -o rvm.sh -L https://get.rvm.io
less rvmsh 
cat rvm.sh | bash -s stable
~~~

完成后, 使得当前终端以使用 RVM 版本的 ruby:
~~~
source ~/.rvm/scripts/rvm
cd ~/git/metasploit-framework
rvm --install $(cat .ruby-version)
~~~

最后, 安装bundle, 以获得你需要的其他gem库:
~~~
gem install bundler
~~~

### 配置 Gnome 终端使用 RVM

Gnome 终端是不聪明的, 并没有让你的 shell  默认情况下是登录型shell, 所以 RVM没有调整配置不能在那里工作。
导航`Edit > Profiles > Highlight Default > Edit > Title and Command > Check [ ] Run command as a login shell.`它看起来像这样，取决于你的特定版本的Gnome：
![](https://github.com/rapid7/metasploit-framework/wiki/screens/kali-gnome-terminal.png)

最后, 请看看您现在正在运行 ruby 版本.
~~~
ruby -v
~~~

如果您运行的ruby 版本仍未是ruby 定义的版本, 您可能需要重新启动终端。如果您的 rvm 最初的安装没有类似于以下内容, 请确保您已将 rvm 添加到您的终端启动中:
~~~
echo [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"' >> .bashrc
~~~
> 看到这,如果没根据指南.用root用户操作的准备好报错吧

### 下载 Bundled Gem
Metasploit有依赖(Ruby 库)。因为您使用的是 RVM, 所以,您可以在本地安装它们, 而不用担心与 Debian 封装的ruby冲突, 这要归功于bundled

~~~
cd ~/git/metasploit-framework/
bundle install
~~~

一两分钟后, 你可以开始使用metasploit了
~~~
./msfconsole
~~~

在第一次启动时顺便创建你的 msf4 目录。
~~~
msfdev@lys:~/git/metasploit-framework$ ./msfconsole
    [*] Starting the Metasploit Framework console.../

                    _---------.
                .' #######   ;."
      .---,.    ;@             @@`;   .---,..
    ." @@@@@'.,'@@            @@@@@',.'@@@@ ".
    '-.@@@@@@@@@@@@@          @@@@@@@@@@@@@ @;
      `.@@@@@@@@@@@@        @@@@@@@@@@@@@@ .'
        "--'.@@@  -.@        @ ,'-   .'--"
              ".@' ; @       @ `.  ;'
                |@@@@ @@@     @    .
                ' @@@ @@   @@    ,
                  `.@@@@    @@   .
                    ',@@     @   ;           _____________
                    (   3 C    )     /|___ / Metasploit! \
                    ;@'. __*__,."    \|--- \_____________/
                      '(.,...."/


          =[ metasploit v4.11.0-dev [core:4.11.0.pre.dev api:1.0.0]]
    + -- --=[ 1420 exploits - 802 auxiliary - 229 post        ]
    + -- --=[ 358 payloads - 37 encoders - 8 nops             ]
    + -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

    msf > ls ~/.msf4
    [*] exec: ls ~/.msf4

    history
    local
    logos
    logs
    loot
    modules
    plugins
    msf > exit
~~~
虽然你还没有设置数据库。但这很容易解决

### 安装 PostgreSQL
kali linux 已经有 Postgresql, 所以我们可以使用它。而用于工作的 Ubuntu 和其他 Debian-based 发行, 假设他们有一个等价的 postgresql 包。确保数据库也在系统启动时启动。
~~~
echo 'YOUR_PASSWORD_FOR_KALI' | sudo -kS update-rc.d postgresql enable &&
echo 'YOUR_PASSWORD_FOR_KALI' | sudo -S service postgresql start &&
cat <<EOF> $HOME/pg-utf8.sql
update pg_database set datallowconn = TRUE where datname = 'template0';
\c template0
update pg_database set datistemplate = FALSE where datname = 'template1';
drop database template1;
create database template1 with template = template0 encoding = 'UTF8';
update pg_database set datistemplate = TRUE where datname = 'template1';
\c template1
update pg_database set datallowconn = FALSE where datname = 'template0';
\q
EOF
sudo -u postgres psql -f $HOME/pg-utf8.sql &&
sudo -u postgres createuser msfdev -dRS &&
sudo -u postgres psql -c \
  "ALTER USER msfdev with ENCRYPTED PASSWORD 'YOUR_PASSWORD_FOR_PGSQL';" &&
sudo -u postgres createdb --owner msfdev msf_dev_db &&
sudo -u postgres createdb --owner msfdev msf_test_db &&
cat <<EOF> $HOME/.msf4/database.yml
# Development Database
development: &pgsql
  adapter: postgresql
  database: msf_dev_db
  username: msfdev
  password: YOUR_PASSWORD_FOR_PGSQL
  host: localhost
  port: 5432
  pool: 5
  timeout: 5

# Production database -- same as dev
production: &production
  <<: *pgsql

# Test database -- not the same, since it gets dropped all the time
test:
  <<: *pgsql
  database: msf_test_db
EOF

~~~

在kali Linux 上, postgresql (和任何其他监听服务) 默认情况下是不启用的。这是一个良好的安全和资源预防措施, 但如果你想要使用它, 随时启动它:
~~~
update-rc.d postgresql enable
~~~

接下来, 切换到 postgres 用户执行少量数据库维护以修复默认编码 (在 @ffmike 的要点中提供了有用的信息)。
~~~
sudo -sE su postgres
psql
update pg_database set datallowconn = TRUE where datname = 'template0';
\c template0
update pg_database set datistemplate = FALSE where datname = 'template1';
drop database template1;
create database template1 with template = template0 encoding = 'UTF8';
update pg_database set datistemplate = TRUE where datname = 'template1';
\c template1
update pg_database set datallowconn = FALSE where datname = 'template0';
\q
~~~

#### 创造一个数据库用户msfdev
在postgresql命令行
~~~
createuser msfdev -dPRS              # Come up with another great password
createdb --owner msfdev msf_dev_db   # Create the development database
createdb --owner msfdev msf_test_db  # Create the test database
exit                                 # Become msfdev again
~~~

#### 创造database.yml
现在你切换回原本用户,创造文件`$HOME/.msf4/database.ym`
~~~
# Development Database
development: &pgsql
  adapter: postgresql
  database: msf_dev_db
  username: msfdev
  password: YOUR_PASSWORD_FOR_PGSQL
  host: localhost
  port: 5432
  pool: 5
  timeout: 5

# Production database -- same as dev
production: &production
  <<: *pgsql

# Test database -- not the same, since it gets dropped all the time
test:
  <<: *pgsql
  database: msf_test_db
~~~

下次启动./msfconsole 时, 将创建开发数据库.检查一下
~~~
./msfconsole -qx "db_status; exit"
~~~

### respc
大多数框架测试都使用 rspec。确保它工作
~~~
rake spec
~~~
您应该看到超过9000测试运行, 主要都是绿色点, 少数是黄色的, 没有红色的错误。

### 配置git
#### 使用dev运行
~~~
cd $HOME/git/metasploit-framework &&
git remote add upstream git@github.com:rapid7/metasploit-framework.git &&
git fetch upstream &&
git checkout -b upstream-master --track upstream/master &&
ruby tools/dev/add_pr_fetch.rb &&
ln -sf ../../tools/dev/pre-commit-hook.rb .git/hooks/pre-commit &&
ln -sf ../../tools/dev/pre-commit-hook.rb .git/hooks/post-merge &&
git config --global user.name   "YOUR_USERNAME_FOR_REAL_LIFE" &&
git config --global user.email  "YOUR_USERNAME_FOR_EMAIL" &&
git config --global github.user "YOUR_USERNAME_FOR_GITHUB"
~~~
#### 设置pull参考
如果您想轻松在您的命令行上访问上游请求,-您需要在. git/config 中添加适当的 fetch 引用。以下操作很容易完成:
~~~
tools/dev/add_pr_fetch.rb
~~~

这将为所有远程仓库添加适当的参考, 包括您的。现在, 你可以做一些奇特的事情, 比如:
~~~
git checkout fixes-to-pr-1234 upstream/pr/1234
git push origin
~~~

在github这样做的方法不太容易描述
这一切都可以让你看看其他的的pull请求 , 作出修改, 并发布到自己的分支。反过来, 这将允许你帮助其他人的pull请求 修正或增加。

#### 保持同步
你大部分时间并不想直接提交给主分支。 始终在分支中进行更改，然后合并这些更改。 这使得与上游保持同步并且不会丢失任何本地更改变得很容易。
##### 同步到上游/主分支
不可能更容易了
~~~
git checkout master
git fetch upstream
git push origin
~~~
这也可以保持pull请求与主分支同步，但除非你遇到合并冲突，你不应该经常这样做。 当你最终解决合并冲突时，你需要在推送重新同步的分支时使用--force，因为你的提交历史将在rebase之后有所不同。

> 对于rapid7/master来说，push是从来没有好的，但对于正在进行的分支，对历史的一点点说明不是违法的。

### Msftidy
为了检查你正在编写的任何新模块，你需要一个预先提交和一个合并后的`hook`来运行我们的lint-checker，`msftidy.rb`。 所以，像这样符号链接：
~~~
cd $HOME/git/metasploit-framework
ln -sf tools/dev/pre-commit-hook.rb .git/hooks/pre-commit
ln -sf tools/dev/pre-commit-hook.rb .git/hooks/post-merge
~~~

### 你的名字
最后，如果您想要为Metasploit做出贡献，您至少需要配置您的用户名和电子邮件地址，如下所示：
~~~
git config --global user.name   "YOUR_USERNAME_FOR_REAL_LIFE"
git config --global user.email  "YOUR_USERNAME_FOR_EMAIL"
git config --global github.user "YOUR_USERNAME_FOR_GITHUB"
~~~
你填写的邮件地址需要和你的github账号的邮件地址相同

### 签名提交
我们喜欢签名提交，主要是因为我们害怕伪造[伪造](https://mikegerwitz.com/papers/git-horror-story)。 程序在这里[详细说明](https://github.com/rapid7/metasploit-framework/wiki/Committer-Keys#signing-howto)。 请注意，名称和电子邮件地址必须完全匹配签名密钥上的信息。 鼓励贡献者签署提交，而Metasploit提交者需要在提交请求时签署合并提交。

### 方便的别名
没有几个方便的别名，开发环境设置也将是完整的，但它可以使您的生活更轻松

#### 覆盖已安装的msfconsole
作为开发用户，您可能会不小心尝试使用已安装的Metasploit msfconsole。 由于RVM怎么处理不同的ruby版本和gemset 各种各样的原因，这种方式不能工作。 所以，创建这个别名
~~~
echo 'alias msfconsole="pushd $HOME/git/metasploit-framework && ./msfconsole && popd"' >> ~/.bash_aliases
~~~
如果您正在使用已安装版本和开发版本，则不同的用户帐户是最好的选择。

#### 提示当前的Ruby/Gemset/Branch
这是超级方便的跟踪你现在在哪里的方法。 把它放在〜/ .bash_aliases中。
~~~
function git-current-branch {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1) /'
}
export PS1="[ruby-\$(~/.rvm/bin/rvm-prompt v p g)]\$(git-current-branch)\n$PS1"
~~~

#### 　Git 别名
Git有自己的处理别名的方法 - 无论是在$HOME/.gitconfig还是repo-name/.git/config   与常规shell别名分离。 下面是一些较为便利的。
~~~
[alias]
# An easy, colored oneline log format that shows signed/unsigned status
nicelog = log --pretty=format:'%Cred%h%Creset -%Creset %s %Cgreen(%cr) %C(bold blue)<%aE>%Creset [%G?]'

# Shorthand commands to always sign (-S) and always edit the commit message.
m = merge -S --no-ff --edit
c = commit -S --edit

# Shorthand to always blame (praise) without looking at whitespace changes
b= blame -w

# Spin up a quick temp branch, because git stash is too spooky.
temp = !"git branch -D temp; git checkout -b temp"

# Create a pull request in a web browser from the CLI. Usage: $1 is HISNAME, $2 is HISBRANCH
# Fixes from @kernelsmith, thanks!
pr-url =!"xdg-open https://github.com/$(git config github.user)/$(basename $(git rev-parse --show-toplevel))/pull/new/$1:$2...$(git branch-current) #"
~~~