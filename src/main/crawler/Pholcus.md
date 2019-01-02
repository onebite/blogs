#   从爬虫到
##  前言
爬虫（web crawler），wiki百科给的解析是自动浏览万维网的网络机器人，像Google和百度的搜索引擎。
爬虫，不仅仅在于机器自动，更关注信息的检索和收集。万维网，是一个庞大的资料库，所以爬虫一直都是热门话题，备受关注，
很多程序员或者学习，都会拿这个练手。我们都知道，爬虫知识第一步，更重要的后面的数据运用。
前段时间，马蜂窝的机器评论新闻也闹得挺热闹的。
但这一步，也不好走，很多网站有反爬机制，如何骗过对方，也挺费劲的。
##关于反爬策略
1. 检测http header中user-agent/referer;
2. 检测cookie，多次访问cookie set次数;
3. 单个IP单位时间内访问次数控制，超过阈值，返回max requests；
4. 验证码；
5. 动态页面，关键数据（如验证码）用ajax请求，甚至加密；
6. 机器行为学习分析；

对于第三点，误伤大，很多公司都不舍得用，除非有很好的算法策略，而且意义不大，现在都找代理买IP分布式爬；
第四，ocr一下，分析些验证码，大学的时候，简单玩过下。现在的验证码多种多样的，比如电话语音验证码，还是挺烦的；
第五点，到现在其实根本不用介意，爬虫会使用phantomjs/selenium模拟浏览器，因为使用一般httpclient肯定被反爬。
最后一点，通过机器分析你的行为，成本高。
##框架
目前网上有很多介绍[scrapy](https://doc.scrapy.org/en/latest/intro/overview.html)和[pyspider](http://docs.pyspider.org/en/latest/Quickstart/)的，
而且官网有很详细的介绍，在这里就不重复介绍。说多一句，对程序员来说，要更好地学习什么技术时，最好看官网doc，并仔细研究源码，学习起来才能事半功倍。
之前用过scrapy，主要写spider和item，用起来挺顺手的。揪心的是，那时候还是python2.7, 要使用多线程还得用gevent的时候，而且对中文很不友好。
本文要介绍另一个分布式爬虫框架pholcus([git传送](https://github.com/henrylee2cn/pholcus.git))。
作者是用golang写的，看过他的源码，很赏心悦目的，推荐大家可以研究下，就像你看hashmap源码有个细节hashcode中的移位一样。
看过scrapy源码的人，可能会说似曾相识吧。
同时，推荐下golang，都说golang晦涩难懂，通过这些年学习和使用C#、java、Python、perl、golang，js，才发现语言万变不离其宗，学习起来有很多互通之处。
至于为什么推荐，学艺不精，说不好的，说下我使用的感觉。平时工作，用Java比较多，当你切换到golang模式，有channel，有lock，有goroutine，
而你只需关心struct和interface，最后build生成可执行文件， 很省心。
pholcus，基本可以拿来即用，你只需写好spider放到pholcus_lib下，并注册即可，具体操作指引请看作者doc。
##新增selenium下载器
pholcus有两个下载器，surf和phantomjs(phantomjs已停止更新)，用起来发现phantomjs还是有很多请求被反爬。
所以在GitHub找了个[chromedp](https://github.com/chromedp/chromedp)还能用，不过它还是用打开Chrome，很多费劲。
而且作者也说了该版本还有待完善，并且很快出来新版本，我们就期待下吧。如果你要使用chromedp, 你可以参考以下代码
`golang

    ctx,cancelF := context.WithCancel(context.Background())
	cdp,err := chromedp.New(ctx)
	if err != nil {
		log.Printf("[E] chromedp init error: %v",err)
	}

	self.Cdp = cdp
	self.ctx = &ctx
	self.cacel = cancelF
	self.initial = true
`

记得自己get

> go get -u github.com/chromedp/chromedp

最后采用webdriver，自己新增了个selenium downloader,效果挺好，爆破力极强，[源码传送](https://github.com/onebite/pholcus/blob/master/app/downloader/surfer/sel.go).
贴一下，chromedriver.exe支持的[webdriver命令](https://chromium.googlesource.com/chromium/src/+/master/docs/chromedriver_status.md)
