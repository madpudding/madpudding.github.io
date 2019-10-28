---
title: "scrapy 忽略请求"
date: 2019-10-28T15:39:02+08:00
draft: false
---
### 背景:有时候在scrapy中,我们经常会碰到一些我们并不需要的url请求,一是它会带来脏数据,而是它会占用请求资源,造成不必要浪费.
### 解决办法:碰到这种情况,我们可以自己写一个过滤这些请求的中间件(middleware).
#### 比如,我的这个是它会在url里面有些关键字,当url中有这些关键字的时候,中间件就会把请求过滤掉.这是这个middleware的设计思路,下面来看具体怎么去实现它:

```python
# 需要用到的包
import re
from scrapy.exceptions import IgnoreRequest

# 中间件代码
class UrlFilterMiddleware:
    """url filter middleware written by fcj"""
    @classmethod
    def from_crawler(cls, crawler):
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    def process_request(self, request, spider):
        from urllib.parse import urlsplit  # 局部区域引用包
        black_set = {   # 关键字
            'beanie', 'brief', 'choker', 'hat', 'scraf', 'pajamas', 'slippes'
        }
        
        split_result = urlsplit(url=request.url).path  
        # 使用urlsplit切割url,并最后得到path,path一般都包含了关键字.
        split_set = set(re.split(r'[/_.-]', split_result))  
        # 再使用正则表达式,切割字符串,得到一个字符串列表,并set.
        intersection = split_set & black_set
        # set求和,若结果的set长度大于0,则两个set间存在相同元素.这一步主要是避免迭代寻找共同元素.
        if len(intersection) > 0:
            raise IgnoreRequest('url filter')
            # 当有共同元素的时候,则证明url里有关键字,抛出忽略请求的异常即可.
            # 官文对这个异常的描述:This exception can be raised by the Scheduler or 
            # any downloader middleware to indicate that the request should be ignored.
        else:
            return None
            # 当返回None值时,爬虫会继续对request做处理.

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)
```
