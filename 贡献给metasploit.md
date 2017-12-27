## 为metasploit做出贡献
### 喜欢黑客的东西？从这里开始。
每隔一段时间，我们都会得到一个要求：“嗨，我是Metasploit的新手，我想帮忙！” 通常的答案是这样的：“太棒了，这里是我们的[bug跟踪器](https://github.com/rapid7/metasploit-framework/issues)，快速开始”

> 原文get crackin　应该是拼写错误


然而，处理Metasploit Framework核心错误或者特别是疯狂的漏洞利用可能不适合新的贡献者。相信我，每个人都是新手，一点都不值得羞愧。这些bug和漏洞通常是复杂的，微妙的，有太多的选择，所以很难开始。这里有一些想法让你开始。


### [CONTRIBUTING.md](https://github.com/rapid7/metasploit-framework/blob/master/CONTRIBUTING.md)
Metasploit是黑客的工具，但是维护他的黑客也恰好是软件工程师。因此，我们在[CONTRIBUTING.md](https://github.com/rapid7/metasploit-framework/blob/master/CONTRIBUTING.md)中有一些希望容易记住的做法和不该做的做法。阅读这些。


#### 服务器漏洞利用
服务器漏洞总是需要; 因为当你可以直接去看一个脆弱的网络的弱点的时候，何必要进行复杂的社会工程活动。以下是一些搜索查询，以帮助您入门：

* 来自[Exploit-DB](http://www.exploit-db.com/remote/)的远程exploit


#### 客户端漏洞
客户端漏洞通常作为远程客户端将连接到的“恶意服务”运行。他们几乎总是需要某种用户交互才能触发，例如查看网页，下载文件或者连接到攻击者控制的服务来进行攻击。

* 　来自通过Google搜索的[SecurityFocus](https://www.google.com/#bav=on.2,or.r_cp.r_qf.&q=site:securityfocus.com+%22Firefox%22+OR+%22Internet+Explorer%22+OR+%22Chrome%22+OR+%22Safari%22+OR+%22Opera%22+-%22Retired%22&safe=off)的浏览器漏洞


#### 本地和特权提升漏洞

特权升级漏洞通常要求攻击者在目标计算机上拥有一个帐户。他们几乎总是被实现为Metasploit exploit模块在一个[本地](https://github.com/rapid7/metasploit-framework/tree/master/modules/exploits/windows/local)树下（依赖于平台），但是有时候他们最好是post模块。尤其是特权升级漏洞

* [Exploit-DB](http://www.exploit-db.com/local/)的本地漏洞

###  不稳定的模块
想要拿起别人离开的？好极了！只需检查救援[不稳定模块](https://github.com/rapid7/metasploit-framework/wiki/Unstable-Modules)的指南，并通过正规测试和代码清理将这些可怜的和不受欢迎的模块完成


#### 框架错误和功能


如果利用开发不是你的事情，但更直接的Ruby开发是，那么这里有一些好的开始：

* [最近的错误](https://github.com/rapid7/metasploit-framework/issues?q=is%3Aissue+is%3Aopen+label%3Abug)，往往是非常容易或很难解决（不是很多中间的）。
* [功能要求](https://github.com/rapid7/metasploit-framework/issues?q=is%3Aissue+is%3Aopen+label%3Afeature)，通常一样。
沿着这些相同的路线是常年需要更好的自动化测试，在规格目录下。
如果你有一个探索奇怪和美妙的代码库的天赋，挑出一个Metasploit核心代码块，并定义你期望的工作行为。
这个搜索是一个理想的开始; 将该错误描述为一个未决的Rspec测试，引用该错误，然后一旦它得到修复，我们将有一个测试。

#### 没有代码
嘿，我们总是可以使用更好的文档。那些在进攻安全部门的人在[Metasploit Unleashed](http://www.offensive-security.com/metasploit-unleashed/Main_Page)方面做得很好，但是就像所有复杂的工作一样，肯定会有错误被发现。如果您有关于如何使Metasploit的文档更清晰，更容易被更多人访问的想法，请坚持下去。

在你的分支写维基文章（提示，[咕噜](https://github.com/gollum/gollum)是非常好的），让别人知道他们，我们很乐意反映在这里，并保留您的荣誉。

与YouTube的屏幕录像一样，您也可以使用特定的常见任务。当你做这件事的时候，叙述是很棒的，看起来好像喜欢这些东西的[YouTube视频](http://www.youtube.com/results?search_query=metasploit&oq=metasploit) .
这里面超过4万，我们希望有人能够加强和管理这些东西的前10名或前100名.我们可以在这里为新的和有经验的用户推广。

对于开发人员类型：我们正在慢慢地将所有的Metasploit转换为使用YARD的标准化注释，所以我们总是可以使用更准确，更全面的YARD文档来找到所有的库。我们将乐意接受只包含注释文档的请求。

再次，Freenode的#metasploit上总是有位置。帮助那里的问题，人们更喜欢在未来帮助你。

> ＃metasploit　irc频道

### 常见警告

您可能不应该在您在意的网络上的机器上运行您在互联网上找到的概念漏洞代码证明。这通常被认为是一个坏主意。你也可能不应该使用你通常的计算机作为攻击exploit的目标，因为你是故意诱发不稳定的行为。

我们首选的模块提交方法是通过你在Metasploit自己的分支上的功能分支的git pull请求。你可以在这里学习如何创建一个 https://github.com/rapid7/metasploit-framework/wiki/Landing-Pull-Requests


另外，请仔细阅读我们了解如何使用git和我们的新模块接受的指南，以防您不熟悉它们：https://github.com/rapid7/metasploit-framework/wiki

如果您遇到困难，请尽可能在我们的Freenode IRC频道#metasploit（加入需要一个注册昵称）上解释您的具体问题。有人应该能够伸出援手。显然，有些人从来不睡觉。