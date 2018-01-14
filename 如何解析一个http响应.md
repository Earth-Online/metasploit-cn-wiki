本文档讨论如何以最简洁的方式解析HTTP响应主体.

### 得到一个响应
要获得响应，您可以使用 Rex::Proto::Http::Client, 或 the HttpClient mixin来生成一个http请求.如果你正在编写一个模块，你应该使用mixin
以下是使用HttpClient中的#send_request_cgi方法的示例：
~~~
res = send_request_cgi({'uri'=>'/index.php'})
~~~
返回值res是一个Rex::Proto::Http::Response 对象，但是也可能由于连接/响应超时而得到一个NilClass。

### 得到一个响应主体
一个Rex::Proto::Http::Response对象,下面是如何检索HTTP正文
~~~
data = res.body
~~~
如果你想获得原始的http响应(包括响应信息,代码,头,主体).你可以简单的
~~~
raw_res = res.to_s
~~~
但是在这个文档,我们只关注主体


### 选择正确的解析器

|   格式 |解析器    |
| --- | --- |
| HTML   | Nokogiri   |
|XML|Nokogiri|
|JSON|JSON|
如果您需要解析的格式不在列表中，则返回res.body

#### 用Nokogiri解析HTML
当你有一个Rex::Proto::Http::Response 的时候，调用的方法是：
~~~
html = res.get_html_document
~~~
这会给你一个Nokogiri::HTML::Document,它将允许你使用Nokogiri api
Nokogiri有两种常用的方法来查找元素：#at和#search。主要区别是#at方法将只返回第一个结果，而#search将返回所有找到的结果（在一个数组中）。

考虑下面的例子作为你的HTML响应：
~~~
<html>
<head>
	<title>Hello, World!</title>
</head>
<body>
	<div class="greetings">
		<div id="english">Hello</div>
		<div id="spanish">Hola</div>
		<div id="french">Bonjour</div>
	</div>
</body>
<html>
~~~

#### #at的基本例子
如果使用#at方法来查找DIV元素
~~~
html = res.get_html_document 
greeting = html.at（' div '）
~~~

然后，greeting变量应该是一个Nokogiri::XML::Element对象，它给了我们这个HTML块（因为#at方法只返回第一个结果)
~~~
<div class="greetings">
<div id="english">Hello</div>
<div id="spanish">Hola</div>
<div id="french">Bonjour</div>
</div>
~~~
从特定元素树中抓取元素
~~~
html = res.get_html_document
greeting = html.at('div//div')
~~~
然后，该greeting变量应该给我们这个HTML块：
~~~
<div id="english">Hello</div>
~~~

#### 抓取具有特定属性的元素
例如我们不想要英文的,我们想要西班牙语的.那我们可以这样做
~~~
html = res.get_html_document
greeting = html.at('div[@id="spanish"]')
~~~

#### 抓取具有特定文本的元素
假设我只知道有一个DIV元素说“Bonjour”，我想抓住它，然后我可以这样做：
~~~
html = res.get_html_document
greeting = html.at('//div[contains(text(), "Bonjour")]')
~~~
或者让我们说，我不知道“Bonjour”这个词是什么元素，那么我可以对此有点模糊
~~~
html = res.get_html_document
greeting = html.at('[text()*="Bonjour"]')
~~~

### #search的基本用法
#search方法返回一个元素数组。假设我们想要找到所有的DIV元素，那么这里是怎么做
~~~ruby
html = res.get_html_document
divs = html.search('div')
~~~

#### 获取文本
当你有一个元素时，你总是可以调用#text方法来获取文本。例如
~~~ruby
html = res.get_html_document
greeting = html.at('[text()*="Bonjour"]')
print_status(greeting.text)
~~~
#text方法也可以被用作去除所有HTML标签
~~~ruby
html = res.get_html_document
print_line(html.text)
~~~
以上将输出：
~~~
"\n\nHello, World!\n\n\n\nHello\nHola\nBonjour\n\n\n"
~~~
如果你是想保留HTML标签，那么不要调用#text，而是调用#inner_html。

#### 访问属性
使用一个元素，只需调用#attributes即可。

#### dom树

使用#next方法转到下一个元素。

使用#previous方法回到前一个元素。

使用#parent方法来查找父元素。

使用#children方法获取所有的子元素。

使用#traverse方法进行复杂的解析。

### 解析xml
要从Rex::Proto::Http::Response获取XML正文，请执行
~~~
xml = res.get_xml_document
~~~
其余的应该和解析HTML非常相似。

### 解析json
要从Rex::Proto::Http::Response获取json正文，请执行
~~~
json = res.get_json_document
~~~