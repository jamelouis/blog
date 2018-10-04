---
layout: post
title:  "json loads keep order"
date:   2017-12-02 22:03:31 +0800
categories: web
---

最近在看[豆瓣API快速入门], 输入如下api到浏览器的地址栏，*Enter*后

> https://api.douban.com/v2/book/1220562
>
>{"rating":{"max":10,"numRaters":368,"average":"7.3","min":0} ...

返回值为json, 单行压缩的文本不便于阅读，需要tabify一下。

## Tabify 

python内置了标准库json, json库的dumps函数可以实现tabify功能。过程如下:
从Server上Requert数据，用json.dumps Tabify后, 存盘。

```python
# -*- coding:utf-8 -*-
import json
import urllib.request

def GetDoubanBook2Json(url):
    with urllib.request.urlopen(url) as f:
        cc = json.loads(f.read().decode('utf-8'), encoding='utf-8')

        save_file_name = url[url.rfind('/')+1:] + '.json'
        with open(save_file_name, 'w', encoding='utf-8') as f:
            f.write(json.dumps(cc, indent=4, ensure_ascii=False))

if __name__ == '__main__':
    url = 'https://api.douban.com/v2/book/1220562'
    GetDoubanBook2Json(url)
```

## Keep Order

和之前的单行压缩文本对比，发现json的key的顺序发生了变化。Google了一下，[python-json-loads-changes-the-order-of-the-object]给了Solution:

```python
from collections import OrderedDict
data_content = json.loads(input_data.decode('utf-8'), object_pairs_hook=OrderedDict)
```

```python
# -*- coding:utf-8 -*-
import json
from collections import OrderedDict
import urllib.request

def GetDoubanBook2Json(url):
    with urllib.request.urlopen(url) as f:
        # object_pairs_hook = OrderedDict, keep order
        cc = json.loads(f.read().decode('utf-8'), object_pairs_hook = OrderedDict, encoding='utf-8')
        
        save_file_name = url[url.rfind('/')+1:] + '.json'
        with open(save_file_name, 'w', encoding='utf-8') as f:
            f.write(json.dumps(cc, indent=4, ensure_ascii=False))

if __name__ == '__main__':
    url = 'https://api.douban.com/v2/book/1220562'
    GetDoubanBook2Json(url)
```

结果对比如图:

![python-json-loads-keep-order](/assets/images/python/python-json-loads-keep-order/python-json-loads-keep-order.jpg "python-json-loads-keep-order")

## Reference
* [豆瓣API快速入门](https://developers.douban.com/wiki/?title=guide)
* [json-python documentation](https://docs.python.org/3/library/json.html)
* [urllib python3.x](https://docs.python.org/3/library/urllib.request.html#module-urllib.request)
* [python-json-loads-changes-the-order-of-the-object](https://stackoverflow.com/questions/43789439/python-json-loads-changes-the-order-of-the-object)

[豆瓣API快速入门]: https://developers.douban.com/wiki/?title=guide
[python-json-loads-changes-the-order-of-the-object]: https://stackoverflow.com/questions/43789439/python-json-loads-changes-the-order-of-the-object

