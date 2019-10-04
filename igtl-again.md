> 20190722:第一次编写

------

# 写在前面

> - **igotolibrary - github** : [https://github.com/qmppz/igotolibrary](https://github.com/qmppz/igotolibrary)
>  - **上一个版本【我去图书馆-抢座助手】:**[https://blog.csdn.net/RenjiaLu9527/article/details/79871288](https://blog.csdn.net/RenjiaLu9527/article/details/79871288)
> - **sessionid 抓包与指令集帮助：**[https://dwz.cn/DueaPBVD ](https://dwz.cn/DueaPBVD)

   【我去图书馆】、【来选座】这类公众号借助微信的平台提供预约座位的服务，本意是方便大家
   
但是防护验证措施比较弱

由此带来的结果就是总是有人在“闷声发大财”偷偷使用程序抢座，这明显的不公平

以现在的防护验证强度推测，自习室座位越紧张的学校，使用外挂抢座的越多；

只是没有人公开源码

-------

# 抢座主逻辑详解  
## 1. 流程
> 1，抓包，获取通信交互数据
> 2，分析参数，尝试离线构造参数
> 3，py模拟提交测试

##  2. 关键函数
function: ***reserve_a_seat***
```python
# reserve a seat
@utils.catch_exception
def reserve_a_seat(self, libid='', coordinate='', pre_or_today=pre):
   key_seat_page = pre_seatmap_page
   verify_key = verify_key
   url_prefix = url_prefix_pre
   seatmap_pageurl = seatmap_pageurl_pre
   seatmap_page_html = utils.get_response(
       url=seatmap_pageurl, sess=self.sess,
       m_headers=self.CF.M_HEADERS_PRE_RESERVE, m_cookies=self.CF.M_COOKIES, verify_key=verify_key)
   if not seatmap_page_html:
       return False
   soup = BeautifulSoup(seatmap_page_html, 'html.parser')
   hexch_js_code = requests.get([e for e in soup.find_all('script') if
          str(e).find(cache_layout_url) >= 0][0]['src'], verify=False)
   hexch_js_code.encoding='utf8'
   ajax_url = re.compile(r'(?<=[A-Z]\.ajax_get\().*?(?=,)').search(hexch_js_code.text).group(0).replace('AJAX_URL', url_prefix)
   hexch_js_code = re.sub(r'[A-Z]\.ajax_get', 'return %s ; T.ajax_get' % ajax_url, hexch_js_code.text)
   http_hexch_seatinfo = execjs.compile(hexch_js_code).call('reserve_seat', str(libid), coordinate)
   response = self.sess.get(http_hexch_seatinfo, proxies=utils.get_proxy(), headers=self.CF.M_HEADERS_PRE_RESERVE, cookies=self.CF.M_COOKIES, verify=False)
   # return 
   return verify_response(response.text)
```
**步骤流程图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019072212444496.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlbmppYUx1OTUyNw==,size_16,color_FFFFFF,t_70)
## 3. 参数获取
上述流程的**关键**就是如何解析得到 ```hex_code```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722134424150.png)
```html
//抢座 url
https://wechat.v2.traceint.com/index.php/prereserve/save/libid=1234&sESx8nLBnQHh=128,63&yzm=	
```
这是一段加密的 ```hex_code```，这应该是为了修复的上一个版本的爬虫漏洞，而新加的验证字段，每次请求都会变；

但是仔细分析【我去图书馆】首页的 html，可以发现有三个 ```<script```，其中的一个url便是生成 ```hex_code```的 js 文件的 url；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722131515873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlbmppYUx1OTUyNw==,size_16,color_FFFFFF,t_70)
打开这个url，可以**清晰**的看到这个**顾名思义**的函数: 

 **reserve_seat**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019072213194248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlbmppYUx1OTUyNw==,size_16,color_FFFFFF,t_70)
所以只要每次去获取这段加密 js 代码，直接使用```python-execjs```模块执行js即可得到```hex_code```

然后提交get请求即可抢座。

到这里我就有了点疑问：

> ### 为何添加了```hex_code```字段，却又把生成这个字段的函数```reserve_seat()```简单直白清晰的放在旁边?


 **...**
 
希望 【我去图书馆】后台验证不要这么直接，能稍微完善一点；屏蔽掉外挂，让大家公平的抢座！

**为了学习！**

--------

# 最后
文章发布即向我去图书馆的官方分享文章链接进行反馈

工程部署到了微信 《**为了学习**》公众号，仅供体验测试，学习交流，服务启动中...
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722170049654.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlbmppYUx1OTUyNw==,size_16,color_FFFFFF,t_70)

还有什么想说的请留言

--------

# 参考链接
 - **igotolibrary - github** : [https://github.com/qmppz/igotolibrary]
 - **上一个版本【我去图书馆-抢座助手】:**[https://blog.csdn.net/RenjiaLu9527/article/details/79871288](https://blog.csdn.net/RenjiaLu9527/article/details/79871288)
 - **sessionid 抓包与指令集帮助:**[https://dwz.cn/DueaPBVD ](https://dwz.cn/DueaPBVD)


