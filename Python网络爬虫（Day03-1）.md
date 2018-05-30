# Python网络爬虫（Day03-1）

---

* 缓存知乎发现上的链接和页面代码
```
from hashlib import sha1
from urllib.parse import urljoin

import pickle
import re
import requests
import zlib

from bs4 import BeautifulSoup
from redis import Redis


def main():
    # 指定种子页面
    base_url = 'http://www.zhihu.com/'
    seed_url = urljoin(base_url, 'explore')
    # 创建Redis客户端
    client = Redis(host='112.74.172.000', port=7266, password='wangmomo-0000')
    # 设置用户代理（否则访问会被拒绝）
    headers = {'user-agent': 'Baiduspider'}
    # 通过requests模块发送GET请求请求并指定用户代理
    resp = requests.get(seed_url, headers=headers)
    # 创建BeautifulSoup对象并指定lxml作为解析器
    soup = BeautifulSoup(resp.text, 'lxml')
    # 创建正则表达式，只获取/question的内容
    href_regex = re.compile(r'^/question')
    # 查找所有href属性，以/question打头的a标签
    for a_tag in soup.find_all('a', {'href': href_regex}):
        # 获取a标签的href属性值，并组装完整的URL
        href = a_tag.attrs['href']
        full_url = urljoin(base_url, href)
        # 将URL处理成SHA1摘要（长度固定更简短）
        hasher = sha1()
        hasher.update(full_url.encode('utf-8'))
        # URL摘要---field_key
        field_key = hasher.hexdigest()
        # 如果Redis的键'zhihu'对应的hash数据类型中没有URL的摘要，就访问页面并缓存。
        # 通过hexists判断，如果摘要已经 出现过，就不在去下载新的页面
        if not client.hexists('zhihu', field_key):
            html_page = requests.get(full_url, headers=headers).text
            # 对页面进行序列化金额压缩操作
            zipped_page = zlib.compress(pickle.dumps(html_page))
            # 使用hash数据类型保存URL摘要及其对应的页面代码
            client.hset('zhihu', field_key, zipped_page)
    # 显示总共缓存了多少个页面
    print('Total %d question pages found.' % client.hlen('zhihu'))
    #  
    client.expire('zhihu', 86400)
    """
    # redis中查看知乎超级时间 显示结果
    112.74.172.200:7266> ttl zhihu
    (integer) 85647
    """
    
if __name__ == '__main__':
    main()
```

* 执行页面, python执行后,在redis-cli中查看 
```
# 
112.74.172.200:7266> hkeys zhihu
 1) "b80da7d62d21027a8ae3a144b979c12d86fc2e2d"
 2) "cd8926df9571b421b557dab0399b782eaca01754"
 3) "d84966f5e28013142afa29d453fda977071da582"
 4) "8e2ab6800618213b3d1ac4d10f334d17e14b1e5c"
 5) "aa30fb121c50eafc2460fb2c661b9d95b0293a7f"
 6) "bd37cc2821c920bcb3f8028579d0397727426362"
 7) "4cd957acd4744e2d7d23f13e4a09ba5260257ee5"
 8) "79b11bb97ded01f9c577ee512a2d4f2feda1d06e"
 9) "ddb3c892b0102ced7a094272ba2d0388d21fe7ef"
10) "8adde7cf2bf1aab037093f8f6dfc940baf8e59e7"
11) "376f74aa6f8558617c28efcf018f52865d87ad06"
12) "9dfaa58ee26b6ce303db128f1e53ce1a9f783bcc"
13) "45695081ec23bd062a177200d98d15251734ffd9"
14) "37a37a8bd48dc34361b29c1f455f0e6c058d0e9c"
15) "55475133149d581a4a3531cfe5a42ab7562a32e6"
16) "43d08e1373bbaf8618e901af217a93070115f304"
17) "dbac7426dfb9adaf9ebfe591c203a93fd9b5e927"
18) "14cd638855ba44d02be3928b06ac20d2b775cf4d"
19) "090c92a2c6a1c6900f5ba31079b4c42cb1040132"
20) "818029d6b8db3a32bf1c5aa704526fab36406603"
21) "0207bd1578a4ea6ee7ca88bbfbf56b2335bceb8f"
22) "4cbc9a0be26dc091578dbaff070502da2c4f6257"
23) "8a5ff0d825b84ab49999b27d372e888a7396afbb"
24) "ee76a485d772661e6f550edc636ea862b27bea8b"
25) "ea965fd3668f02f8411b2c05afb48c2684942055"
26) "2ca07b7dff20f9f4c4358019eee9ea3bb95925f5"
27) "4a0fecdc2a12c90dc53a0f5e5604fa6f97d3d3bc"
28) "06f620a2cb129be1b24520ffd488004da02aa9a9"
```
* 第二次执行，比第一次快了很多，因为已经缓存了，相比昨天，有15个页面有更新

* expire 给键设置存活时间
```
# 存在10秒
set username admin ex 10
# 获取username,只有 10秒
get username
# ttl（time to live）--存活时间
ttl username
```
* 实验1
```
[root@iZwz9295yl5iuu2cbsoscsZ ~]# redis-cli -h 112.74.172.237 -p 7266
112.74.172.200:7266> auth wangmomo-0726
OK
112.74.172.200:7266> get username
(nil)  # 没有
112.74.172.200:7266> ttl username
(integer) -2 # 已经过期
112.74.172.200:7266> 
```
* 实验2
```
112.74.172.200:7266> set username momo
OK
112.74.172.200:7266> ttl username
(integer) -1 # 没有设置超期时间(ttl)
112.74.172.200:7266> expire  username 20 # 超期时间为20秒
(integer) 1
112.74.172.200:7266> ttl username
(integer) 13
112.74.172.200:7266> ttl username
(integer) 9
112.74.172.200:7266> ttl username
(integer) 7
```

```
# slaveof <masterip> <masterport>
```