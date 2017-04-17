## 淘宝爬虫需要注意事项
---
最近在做一个关于淘宝同款的爬虫，即搜索与淘宝某个类似商品的同款商品，并记录下来这些同款商品的图片。商品由一个文件提供，里面是一些web的访问日志， 需要从里面提取出uniqueid，是一串数字，然后再和淘宝搜同款的url进行拼接，生成url进行访问，然后解析页面内容，提取数据，记录。

看似挺简单的工作，竟然也是踩坑不少，记录下来，下次抓取的时候注意。

1. **速率**
    众所周知，淘宝在反爬虫方面是出了名的严厉。一开始我的速率是3s请求一次，在少量样本的情况下（不到100个）工作良好，给我造成了错觉，以为这个速率是可以的。后来在样本变大的时候，就被无情地封杀了。换台服务器，接着来，4s，然后大约10分钟又是被封杀。由于服务器资源有限，只好速率降成10s一次，工作良好（还好这项任务对速率没有要求）。
2. **http header**
    爬虫自然要伪装成正常浏览器了，所以一开始我几乎把所有请求头都加上了（包括登陆过后的cookie）。但是过了一会就发现请求的页面被跳转了，需要登陆。个人猜测应该是服务器那边定时换cookie，导致旧cookie失效。
     山穷水复，柳暗花明。当我一筹莫展之际，我把所有的请求头都去掉，尝试一下，结果竟然成功抓取了！
3. **页面解析、下一页**
    同款页面返回的数据并没有在html标签里，而是在js里，然后在浏览器端经过计算，填充到应该在的标签里。在`python`如果不用`phantomjs`的话，就无法解析html结构进行抓取了。好在数据是json格式的，从返回的html文本中用正则表达式提取出json，然后再loads为python的数据结构，取出想要的元素（我这里是pic_url）
经过观察，同款的下一页是直接在url后面加个`&s=60`这样的参数代表第二页从第60个商品开始。第三页自然就是`&s=120`了。但是如果想要得到页数，还要在上面返回的json中进行查找。我想到了一个比较简单的方法，利用`python`的`itertools.count`生成器，生成从0开始的整数，然后乘以60，进行url拼接。假如到某一页停止了，无法解析出`pic_url`字段，会产生异常，然后就从这里跳出本次循环，进行下一个uniqid的遍历
4. **uniqueid的去重，暂存**
    这是我第一次没有考虑到的。我没有仔细看提供样本的文件，只知道有很多数字。后来第二遍看的时候，发现里面有好多重复的，应首先去重。当然去重之后还是有好多数据。我就想，如果爬虫异常，重新开始，还是需要从头开始进行抓取，本来爬虫就不快，这要等到何年何月？好在`python`有针对其内置类型的序列化模块，可以将其保存在文件里。对于`uidlist`，我将其序列化到文件上，然后爬虫开始的时候再反序列化这个文件，取出`uidlist`，对于其中的元素，抓取成功后将其从`uidlist`中删除，再将`uidlist`序列化为文件。这样可以保证下次开始的时候之前的都已抓取过，无需重新抓取。

抓取代码如下：
```python
#coding:utf8
import time
import re
import json
import itertools
import sys
import requests
from itertools import count
import traceback
import cPickle


def json_handler(uid,json_dict):
    if not json_dict:
        return False
    try:
        item_list = json_dict['mods']['recitem']['data']['items']
        pic_set = set()
        with open('new_result.log','a') as f:
            f.write(uid[1:]+'\n')
            for item in item_list:
                pic_set.add(item['pic_url'])
            for pic in pic_set:
                f.write(pic+'\n')
        return True
    except KeyError:
        print 'None'
        return False


def parse_url(u):
    try:
        res = requests.get(u).text
        json_str = re.findall(' = {.*};', res)[0][3:-1]
        json_dict = json.loads(json_str)
        return json_dict
    except:
        return None

#读取pickle文件
with open(r'pickle_file') as pf:
    uidlist = cPickle.load(pf)

while uidlist:
    try:
        uid = uidlist[0]
        url ="http://s.taobao.com/search?type=samestyle&app=i2i&rec_type=&uniqpid=%s&s=0"%uid
        num = count(0)
        while True:
            qstring = 60*num.next()
            new_url = url.replace('&s=0','&s='+str(qstring))
            print new_url
            try:
                jd = parse_url(new_url)
            except:
                with open('except.log','a') as f:
                    f.write(traceback.format_exc()+'\n'+new_url)
                r = requests.get(new_url).text
                with open('html.html','w') as w:
                    w.write(r.encode('utf8'))
                break
            if not json_handler(uid,jd):
                break
            #将 此uid从uidlist删除，并保存pickle文件
            uidlist.pop(0)
            with open('pickle_file','w') as pf:
                cPickle.dump(uidlist,pf)
            time.sleep(10)
    except:
        pass
        
##another file
with open(r'uniqpid.log') as f:
    content = f.read()
    uidlist = re.findall('-\d+', content)
#去重
uidlist = list(set(uidlist))

with open(r'pickle_file','w') as pf:
    cPickle.dump(uidlist,pf)
```