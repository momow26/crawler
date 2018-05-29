# Python网络爬虫（Day02-3）

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
        field_key = hasher.hexdigest()
        # 如果Redis的键'zhihu'对应的hash数据类型中没有URL的摘要，就访问页面并缓存。
        if not client.hexists('zhihu', field_key):
            html_page = requests.get(full_url, headers=headers).text
            # 对页面进行序列化金额压缩操作
            zipped_page = zlib.compress(pickle.dumps(html_page))
            # 使用hash数据类型保存URL摘要及其对应的页面代码
            client.hset('zhihu', field_key, zipped_page)
    # 显示总共缓存了多少个页面
    print('Total %d question pages found.' % client.hlen('zhihu'))


if __name__ == '__main__':
    main()
```






