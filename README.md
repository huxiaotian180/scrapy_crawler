# scrapy_crawler
scrapy分布式爬虫框架


![](https://github.com/huxiaotian180/ImageCache/raw/master/Loge/scrapy.png)


Spider是最核心的组件，Scrapy爬虫开发是围绕实现Spider展开的

Request和Response是HTTP协议中的术语，即HTTP请求和HTTP响应，Scrapy框架中定义了相应的Request和Response类，这里的Item代表Spider从页面中爬取的一项数据



●　当SPIDER要爬取某URL地址的页面时，需使用该URL构造一个Request对象，提交给ENGINE（图2-1中的1）。

●　Request对象随后进入SCHEDULER按某种算法进行排队，之后的某个时刻SCHEDULER将其出队，送往DOWNLOADER（图2-1中的2、3、4）。

●　DOWNLOADER根据Request对象中的URL地址发送一次HTTP请求到网站服务器，之后用服务器返回的HTTP响应构造出一个Response对象，其中包含页面的HTML文本（图2-1中的5）。

●　Response对象最终会被递送给SPIDER的页面解析函数（构造Request对象时指定）进行处理，页面解析函数从页面中提取数据，封装成Item后提交给ENGINE，Item之后被送往ITEM PIPELINES进行处理，最终可能由EXPORTER（图2-1中没有显示）以某种数据格式写入文件（csv，json）；另一方面，页面解析函数还从页面中提取链接（URL），构造出新的Request对象提交给ENGINE（图2-1中的6、7、8）



Request和Response对象

Request

Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback])

●　url（必选）

请求页面的url地址，bytes或str类型，如'http://www.python.org/doc'。

●　callback

页面解析函数， Callable类型，Request对象请求的页面下载完成后，由该参数指定的页面解析函数被调用。如果未传递该参数，默认调用Spider的parse方法。

●　method

HTTP请求的方法，默认为'GET'。

●　headers

HTTP请求的头部字典，dict类型，例如{'Accept': 'text/html', 'User-Agent':Mozilla/5.0'}。如果其中某项的值为None，就表示不发送该项HTTP头部，例如{'Cookie': None}，禁止发送Cookie。

●　body

HTTP请求的正文，bytes或str类型。

●　cookies

Cookie信息字典，dict类型，例如{'currency': 'USD', 'country': 'UY'}。

●　meta

Request的元数据字典，dict类型，用于给框架中其他组件传递信息，比如中间件Item Pipeline。其他组件可以使用Request对象的meta属性访问该元数据字典（request.meta），也用于给响应处理函数传递信息，详见Response的meta属性。

●　encoding

url和body参数的编码默认为'utf-8'。如果传入的url或body参数是str类型，就使用该参数进行编码。

●　priority

请求的优先级默认值为0，优先级高的请求优先下载。

●　dont_filter

默认情况下（dont_filter=False），对同一个url地址多次提交下载请求，后面的请求会被去重过滤器过滤（避免重复下载）。如果将该参数置为True，可以使请求避免被过滤，强制下载。例如，在多次爬取一个内容随时间而变化的页面时（每次使用相同的url），可以将该参数置为True。

●　errback

请求出现异常或者出现HTTP错误时（如404页面不存在）的回调函数。



import scrapy

request2 = scrapy.Request('http://quotes.toscrape.com/', callback=self.parseItem)

Response


Response对象用来描述一个HTTP响应，Response只是一个基类，根据响应内容的不同有如下子类：

●　TextResponse

●　HtmlResponse

●　XmlResponse

当一个页面下载完成时，下载器依据HTTP响应头部中的Content-Type信息创建某个Response的子类对象。我们通常爬取的网页，其内容是HTML文本，创建的便是HtmlResponse对象，其中HtmlResponse和XmlResponse是TextResponse的子类。实际上，这3个子类只有细微的差别，这里以HtmlResponse为例进行讲解

下面介绍HtmlResponse对象的属性及方法。

●　url

HTTP响应的url地址，str类型。

●　status

HTTP响应的状态码，int类型，例如200，404。

●　headers

HTTP响应的头头部，类字典类型，可以调用get或getlist方法对其进行访问，例如：response.headers.get('Content-Type') response.headers.getlist('Set-Cookie')

●　body

HTTP响应正文，bytes类型。

●　text

文本形式的HTTP响应正文，str类型，它是由response.body使用response.encoding解码得到的，即

reponse.text = response.body.decode(response.encoding)

●　encoding

HTTP响应正文的编码，它的值可能是从HTTP响应头部或正文中解析出来的。

●　request

产生该HTTP响应的Request对象。

●　meta

即response.request.meta，在构造Request对象时，可将要传递给响应处理函数的信息通过meta参数传入；响应处理函数处理响应时，通过response.meta将信息取出。

●　selector

Selector对象用于在Response中提取数据（选择器相关话题在后面章节详细讲解）。

●　xpath(query)

使用XPath选择器在Response中提取数据，实际上它是response.selector.xpath方法的快捷方式（选择器相关话题在后面章节详细讲解）。

●　css(query)

使用CSS选择器在Response中提取数据，实际上它是response.selector.css方法的快捷方式（选择器相关话题在后面章节详细讲解）。

●　urljoin（url）

用于构造绝对url。当传入的url参数是一个相对地址时，根据response.url计算出相应的绝对url。例如，response.url为http://www.example.com/a，url为b/index.html，调用response.urljoin(url)的结果为http://www.example.com/a/b/index.html



虽然HtmlResponse对象有很多属性，但最常用的是以下的3个方法：

●　xpath(query)

●　css(query)

●　urljoin(url)

前两个方法用于提取数据，后一个方法用于构造绝对url
