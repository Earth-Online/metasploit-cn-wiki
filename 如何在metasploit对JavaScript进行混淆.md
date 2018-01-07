## 如何在metasploit对JavaScript进行混淆
隐藏是在开发过程中考虑的一个重要特点.如果你的exploit永远被抓住.那么这和你的exploit多棒或者多么有技术挑战是不重要的.在真正的渗透测试中，它很可能不是很有用。
特别是浏览器漏洞,主要依靠JavaScript来触发漏洞,因此许多基于防病毒或基于特征的入侵检测/防御系统将扫描JavaScript并将特定行标记为恶意代码.以下代码曾被多个防病毒软件供应商视为MS12-063，即使这些代码不一定有害或恶意，我们将在整个wiki上使用它作为示例：

~~~
var arrr = new Array();
arrr[0] = windows.document.createElement("img");
arrr[0]["src"] = "a";
~~~
为了避免被标记,我们可以尝试一些常见的躲避技巧.例如,您可以手动修改代码,使其不能被任何签名识别.或者，如果防病毒软件依靠缓存的网页来扫描漏洞，则可能导致浏览器不缓存你的漏洞网页，以避免漏洞检查。或者在这种情况下,你可以混淆你的代码，这是这篇文章的重点。

在Metasploit中，有三种常见的方法来混淆你的JavaScript。第一个是简单地使用rand_text_alpha方法（在Rex）随机化你的变量。第二个是使用ObfuscateJS类。第三个选项是JSObfu类。

### rand_text_alpha 技巧
使用rand_text_alpha 是最基本的逃避技巧.但也是最没有效的。如果这是你的选择，你应该随机化任何可以随机化，而不会破坏代码。
通过使用上面的MS12-063，你将学会如何使用rand_text_alpha
~~~
# Randomizes the array variable
# Max size = 6, Min = 3
var_array = rand_text_alpha(rand(6) + 3)

# Randomizes the src value
val_src   = rand_text_alpha(1)

js = %Q|
var #{var_array} = new Array();
#{var_array}[0] = windows.document.createElement("img");
#{var_array}[0]["src"] = "#{val_src}";
|
~~~

### ObfuscateJS 类
ObfuscateJS类就像rand_text_alpha,但更好。它允许您替换符号名称，如变量，方法，类和名称空间。它也可以通过随机使用fromCharCode或unescape来混淆字符串.最后，它可以去掉JavaScript注释，这是非常方便的，因为漏洞经常难以理解和阅读，所以您需要注释来记住为什么要以特定的方式编写某些内容，但是您不想在渗透中显示或泄露这些注释
为了使用ObfuscateJS，我们再次使用MS12-063的例子来演示。如果你觉得自己不用编写模块就可以自己完成这个步骤，你可以做的就是继续运行msfconsole，然后切换到irb，如下所示：
~~~
$ ./msfconsole -q
msf > irb
[*] Starting IRB shell...

>> 
~~~

你准备好后.你使用ObfuscateJS做的第一件事是你需要用你想混淆的JavaScript来初始化它，所以在这种情况下，就像下面这样开始：
~~~
js = %Q|
var arrr = new Array();
arrr[0] = windows.document.createElement("img");
arrr[0]["src"] = "a";
|

obfu = ::Rex::Exploitation::ObfuscateJS.new(js)
~~~
obfu应该是一个返回的Rex::Exploitation::ObfuscateJS对象.它允许你做很多事情,你可以真正的调用方法,或者查看源代码，看看有什么方法可用(附加的API文档)。但是为了演示目的，我们将展示最常用的一个：obfuscate方法。

要实际混淆,您需要调用该obfuscate方法.这个方法接受一个符号参数,它允许你手动指定要混淆的符号名(变量,方法,类等等),它应该像下面这样
~~~
{
	'Variables'  => [ 'var1', ... ],
	'Methods'    => [ 'method1', ... ],
	'Namespaces' => [ 'n', ... ],
	'Classes'    => [ { 'Namespace' => 'n', 'Class' => 'y'}, ... ]
}
~~~
所以，如果我想混淆变量arrr，我想混淆src字符串，这是怎么做
~~~
>> obfu.obfuscate('Symbols' => {'Variables'=>['arrr']}, 'Strings' => true)
=> "\nvar QqLFS = new Array();\nQqLFS[0] = windows.document.createElement(unescape(String.fromCharCode(  37, 54, 071, 045, 0x36, 0144, 37, 066, 067 )));\nQqLFS[0][String.fromCharCode(  115, 0x72, 0143 )] = unescape(String.fromCharCode(  045, 0x36, 0x31 ));\n"
~~~
在某些情况下，您实际上可能想知道符号名称的混淆版本。一种情况是从元素的事件处理程序调用JavaScript函数，例如
~~~
<html>
<head>
<script>
function test() {
	alert("hello, world!");
}
</script>
</head>
<body onload="test();">
</body>
</html>
~~~
混淆的版本如下所示
~~~
js = %Q|
function test() {
	alert("hello, world!");
}
|

obfu = ::Rex::Exploitation::ObfuscateJS.new(js)
obfu.obfuscate('Symbols' => {'Methods'=>['test']}, 'Strings' => true)

html = %Q|
<html>
<head>
<script>
#{js}
</script>
</head>
<body onload="#{obfu.sym('test')}();">
</body>
</html>
|

puts html
~~~


### JSObfu类
JSObfu类曾经是ObfuscateJS的堂兄弟，但自2014年9月以来已经完全重写，并被封装成一个gem。混淆更为复杂，您实际上可以用它混淆多次。您也不再需要手动指定要更改的符号名称，它是知道的。

#### 尝试jsobfu
让我们再回到irb来展示使用JSObfu是多么容易
~~~
$ ./msfconsole -q
msf > irb
[*] Starting IRB shell...

>> 
~~~
这个时候我们尝试`hello world`的例子
~~~
>> js = ::Rex::Exploitation::JSObfu.new %Q|alert('hello, world!');|
=> alert('hello, world!');
>> js.obfuscate
=> nil
~~~
它的输出是这样的
~~~
window[(function () { var _d="t",y="ler",N="a"; return N+y+_d })()]((function () { var f='d!',B='orl',Q2='h',m='ello, w'; return Q2+m+B+f })());
~~~
像ObfuscateJS一样，如果你需要得到一个符号名称的随机版本，你仍然可以这样做。我们将用下面的例子来演示
~~~
>> js = ::Rex::Exploitation::JSObfu.new %Q|function test() { alert("hello"); }|
=> function test() {
  alert("hello");
}
>> js.obfuscate
~~~
如果我们想知道方法名test的随机版本
~~~
>> puts js.sym("test")
~~~
很好,快速确认一下
~~~
>> puts js
function _(){window[(function () { var N="t",r="r",i="ale"; return i+r+N })()](String.fromCharCode(0150,0x65,0154,0x6c,0x6f));}
~~~
我看起来很好,最后，让我们尝试混淆几次，看看结果如何：
~~~
>> js = ::Rex::Exploitation::JSObfu.new %Q|alert('hello, world!');|
=> alert('hello, world!');
>> js.obfuscate(:iterations=>3)
=> window[String[((function(){var s=(function () { var r="e"; return r })(),Q=(function () { var I="d",dG="o"; return dG+I })(),c=String.fromCharCode(0x66,114),w=(function () { var i="C",v="r",f="omCh",j="a"; return f+j+v+i })();return c+w+Q+s;})())](('Urx'.length*((0x1*(01*(1*020+5)+1)+3)*'u'.length+('SGgdrAJ'.length-7))+(('Iac'.length*'XLR'.length+2)*'qm'.length+0)),(('l'.length*((function () { var vZ='k'; return vZ })()[((function () { var E="h",t="t",O="leng"; return O+t+E })())]*(0x12*1+0)+'xE'.length)+'h'.length)*(function () { var Z='uA',J='tR',D='x'; return D+J+Z })()[((function () { var m="th",o="g",U="l",Y="en"; return U+Y+o+m })())]+'lLc'.length),('mQ'.length*(02*023+2)+('Tt'.length*'OEzGiMVf'.length+5)),(String.fromCharCode(0x48,0131)[((function () { var i="gth",r="len"; return r+i })())]*('E'.length*0x21+19)+(0x1*'XlhgGJ'.length+4)),(String.fromCharCode(0x69)[((function () { var L="th",Q="n",$="l",I="g",x="e"; return $+x+Q+I+L })())]*('QC'.length*0x2b+3)+(01*26+1)))]((function(){var C=String[((function () { var w="rCode",j="mCha",A="fr",B="o"; return A+B+j+w })())]((6*0x10+15),('riHey'.length*('NHnex'.length*0x4+2)+4),(01*95+13),(1*('Z'.length*(0x1*(01*(0x3*6+5)+1)+18)+12)+46),(0x1*(01*013+6)+16)),JQ=String[((function () { var NO="ode",T="rC",HT="fromCha"; return HT+T+NO })())](('J'.length*0x54+17),(0x2*051+26),('TFJAGR'.length*('ymYaSJtR'.length*'gv'.length+0)+12),(01*0155+2),(0xe*'FBc'.length+2),(0x1*22+10),(3*(01*043+1)+11)),g=(function(){var N=(function () { var s='h'; return s })();return N;})();return g+JQ+C;})());
~~~
### 使用jsbofu进行模块开发
当你正在编写一个模块时，你不应该像上面的例子那样直接调用Rex。相反，你应该使用JSObfu mixin中的#js_obfuscate方法。当你在你的模块中使用JavaScript时，一定要这样写：
~~~
# This returns a Rex::Exploitation::JSObfu object
js = js_obfuscate(your_code)
~~~
