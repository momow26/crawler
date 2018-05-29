# Python网络爬虫（Day01）

---

#### 网络爬虫

网络爬虫（Web crawler/Spider），是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本，它们被广泛用于互联网搜索引擎或其他类似网站，可以自动采集所有其能够访问到的页面内容，以获取或更新这些网站的内容和检索方式。从功能上来讲，爬虫一般分为数据采集，处理，储存三个部分。

---

#### 爬虫规则

* 1、Robots协议

```Robots```协议也称作爬虫协议、机器人协议，它的全名叫作网络爬虫排除标准（Robots Exclusion Protocol），用来告诉爬虫和搜索引擎哪些页面可以抓取，哪些不可以抓取。它通常是一个叫作```robots.txt```的文本文件，一般放在网站的根目录下。

搜索爬虫访问一个站点时:
1）、检查此站点的根目录下是否存在```robots.txt```文件；
2）、如果存在，搜索爬虫会根据其中定义的爬取范围来爬取。
3）、如果不存在，搜索爬虫便会访问所有可直接访问的页面。

Robots协议案例：
```
User-agent: * 
Disallow: /
Allow: /public/
```
这实现了对所有搜索爬虫只允许爬取```public```目录的功能，将上述内容保存成```robots.txt```文件，放在网站的根目录下，和网站的入口文件（比如```index.php```、```index.html```和```index.jsp```等）放在一起。

其中```User-agent```表示搜索爬虫的名称，此处的*表示该协议对任何爬取爬虫有效。比如，我们可以设置：
```
User-agent: Baiduspider
```

这就代表我们设置的规则对百度爬虫是有效的。如果有多条```User-agent```记录，则就会有多个爬虫会受到爬取限制，但至少需要指定一条。

本文中的```Disallow```指定了不允许抓取的目录，比如上例子中设置为/则代表不允许抓取所有页面。

本文中的```Allow```一般和```Disallow```一起使用，一般不会单独使用，用来排除某些限制。现在我们设置为/public/，则表示所有页面不允许抓取，但可以抓取```public```目录。

下面我们再来看几个例子。禁止所有爬虫访问任何目录的代码如下：
```
User-agent: * 
Disallow: /
```

允许所有爬虫访问任何目录的代码如下：
```
User-agent: *
Disallow:
```
另外，直接把```robots.txt```文件留空也是可以的。

禁止所有爬虫访问网站某些目录的代码如下：
```
User-agent: *
Disallow: /private/
Disallow: /tmp/
```

只允许某一个爬虫访问的代码如下：
```
User-agent: WebCrawler
Disallow:
User-agent: *
Disallow: /
```
---

### 应用领域

*  1、搜索引擎
*  2、新闻聚合
*  3、社交应用
*  4、舆情监控
*  5、行业数据

### 爬虫项目练习
* 1、原生态爬取数据的写法

* 2、步骤：
1)、获取资源、数据。
2)、对数据进行解析、处理。
3)、对处理后的数据进行存储、持久化。
* 3、

request
它是最基本的```HTTP```请求模块，可以用来模拟发送请求。就像在浏览器里输入网址然后回车一样，只需要给库方法传入URL以及额外的参数，就可以模拟实现这个过程了。


使用```urllib```的```request```模块，我们可以方便地实现请求的发送并得到响应。



1、urlopen（）
urllib.request模块提供了最基本的构造HTTP请求的方法，利用它可以模拟浏览器的一个请求发起过程，同时它还带有处理授权验证（authenticaton）、重定向（redirection)、浏览器```Cookies```以及其他内容。

下面以爬取搜狐网站为例---[搜狐体育-NBA][1]
  [1]: http://sports.sohu.com/nba_a.shtm 
把这个网页抓下来
```
# from urllib.request import urlopen
import urllib.request


def main():
    # read()---读取页面
    html = urllib.request.urlopen('http://sports.sohu.com/nba_a.shtml').read()
    # 注意：搜狐网站的charset格式为GBK
    print(html.decode('gbk'))


if __name__ == '__main__':
    main()
```
运行结果如下：

我们只用了两行代码，即得到了《搜狐体育-NBA》页面的抓取，输出了网页的源代码。随之即可获取---链接、图片地址、文本信息。

利用```type()```方法查看输出响应的类型：
```
import urllib.request


def main():
    html = urllib.request.urlopen('http://sports.sohu.com/nba_a.shtml')
    print(type(html))


if __name__ == '__main__':
    main()
```
显示结果如下：
```
<class 'http.client.HTTPResponse'>
```
可以发现，它是一个```HTTPResposne```类型的对象。它主要包含```read()```、```readinto()```、```getheader(name)```、```getheaders()```、```fileno()```等方法，以及```msg```、```version```、```status```、```reason```、```debuglevel```、```closed```等属性。

得到这个对象之后，我们把它赋值为```response```变量，然后就可以调用这些方法和属性，得到返回结果的一系列信息了。

例如，调用```read()```方法可以得到返回的网页内容，调用```status```属性可以得到返回结果的状态码，如```200```代表请求成功，```404```代表网页未找到等。

下面再通过一个实例来看看：
```
import urllib.request
def main():
    # read()---读取页面
    # html = urllib.request.urlopen('http://sports.sohu.com/nba_a.shtml').read()
    html = urllib.request.urlopen('http://sports.sohu.com/nba_a.shtml')
    # print(html.decode('gbk'))
    print(type(html))
    print(html.status)
    print(html.getheaders())
    print(html.getheader('Server'))
```
可见，前两个输出分别输出了响应的状态码和响应的头信息，最后一个输出通过调用```getheader()```方法并传递一个参数```Server```获取了响应头中的```Server```值，结果是```xxxx```，意思是服务器是用```xxxx```搭建的。

2、提取特定的a标签，获取```href```属性
```
<a test=a href='https://www.sohu.com/a/233183487_458722' target='_blank'>詹皇：精疲力竭不想谈决赛 打满48分钟是我要求</a>
```

```
# 获取页面的源代码
"""
text---字符
content---二进制
urllib库
一、***** 获取页面数据
二、解析页面数据
"""
# 导入request模块
# from urllib.request import urlopen
import urllib.request
from urllib.error import URLError
import re

# 重新写一个函数，获取页面的代 <retry_times=3, charset='utf-8'》---命名关键字参数，需要 指定名字
def get_page_code(start_url, *, retry_times=3, charset='utf-8'):
    # 页面不写死，start_url, 重置次数，retry_times=3  read()---读取页面, 指定编码 charset='utf-8'
    try:
        # 解码 ，需要指定编码decode(charset)
        html = urllib.request.urlopen(start_url).read().decode(charset)
        # html = urllib.request.urlopen('http://sports.sohu.com/nba_a.shtml')
    # 捕获异常
    except URLError as e:
        print('Error:', e)
        # 再次尝连接
        if retry_times > 0:
            return get_page_code(start_url, retry_times - 1)
        else:
            return None
    return html
    # 不是所有的 网站都是 utf-8
    # print(html.decode('gbk'))
    # print(type(html))
    # print(html.status)
    # print(html.getheaders())
    # print(html.getheader('Server'))


def main():

    html = get_page_code('http://sports.sohu.com/nba_a.shtml', charset='gbk')
    # 获取到页面
    # print(html)
    # (\s--表示空格  [^>]---除了大于号，其他都可以) + ---一个或多个  * ---零个或多个 \S* ---匹配非空白字符外所有的
    # 捕获组 (.*)
    # 命名捕获组
    link_list = re.findall(r'<a[^>]+test=a\s[^>]*href=["\'](\S*)["\']', html)
    # 匹配所有的a标签的href属性
    # link_list = re.findall(r'<a[^>]+*href=["\'](\S*)["\']', html)

    # 惰性匹配
    # link_list = re.findall(r'<a[^>]+test=a\s[^>]*href=["\'](.*?)["\']', html)
    print(link_list)
    print(len(link_list))


if __name__ == '__main__':
    main()
```