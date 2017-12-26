使用这个资源集合来处理Metasploit框架的git仓库。

* * * * *
* [Git 表](http://1)
* [参考网站](http://2)
* [安装metasploit开发环境](安装metasploit.md) 这将引导您创建一个pull请求
* [开始一个pull请求](http://3)
* [远程分支裁剪](http://1)

分叉就是当你把别人的代码库快照进你自己的储存库，应该是在`github.com`上，而且这个代码库可能有它自己的分支，但你通常快照主分支。
你一般克隆你的分支到你自己的机器，然后创建自己的分支。这是你自己的分叉的旁枝。
这些快照，即使如果你push到你的github仓库，对于原始仓库rapid7/metasploit-framework.也不是其中的一部分。
如果你提交一个pull请求，你的分支(通常)能push到原始代码库的主分支(通常情况下，如果你的代码是一个巨大的变化或者其他东西，那么你可能成为一个实验性分支，但这不是典型例子）。
你只要分叉一次，当你需要代码的机器克隆多次，而且你可以随心所欲地进行分支，提交和push（你不需要总是push，你可以推迟，也可以不推迟，但是在做出拉取请求之前进行pull) .当你准备好提交一个PR请求,查看下面
~~~
github.com/rapid7/metasploit-framework --> fork --> github.com/<...>/metasploit-framework
    ^                                                          |
    |                               git clone git://github.com/<...>/metasploit-framework.git
    |                                                          |
    `-- accepted <-- pull request                              V
                      ^                        /home/<...>/repo/metasploit-framework
                      |                                |              |          |
   github.com/<...>/metasploit-framework/branch_xyz    |              |          |
                      |                                |              V          V
                      |                                V           branch_abc   ...
                      `--       push       <--      branch_xyz
~~~
