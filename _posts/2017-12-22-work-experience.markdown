---
layout: post
title:  "Work Experience"
date:   2017-12-22 22:03:31 +0800
categories: python
---

## 概述 ##

进入职场两年半了，至此年末，想知道这两年半的工作都做了什么？作为一名开发人员，
每天都会用版本管理软件svn,perfoce来提交工作信息。因此，可以
从这些项目版本管理软件中提取出历史的提交信息来作为分析的数据，再根据自身维护的
功能模块的关键词来显示功能提交的时间线。

大致的过程如下：

1. 从svn,p4v提取提交历史到文本文件
2. 归一化数据到统一的csv文件中
3. 图表化提交历史数据

### 从svn,p4v提取提交历史到文本文件 ###

#### 从p4v提取提交历史到文本文件 ####

svn提供了很便捷的log导出命令，导出可以是文本格式或者xml。因为文本格式的数据的结构
没有xml来得清晰，为了便于接下来的数据预处理，这里选择导出格式xml。

```
svn log --xml >> log_svn.xml
```

svn导出的xml文件的内容大致如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<log>
<logentry
   revision="1">
<author>zhangsan</author>
<date>2017-12-13T14:06:33.374887Z</date>
<msg>hello,world</msg>
</logentry>
...
</log>
```

#### 从p4v提取提交历史到文本文件 ####
Google了一下，p4v的提交历史可以用如下的命令导出。

``` p4v
p4 changelists -l
```

p4v导出的信息如下：
```
Change 1 on 2017/12/14 by zhangsan@email.com

	hello,world
```

### 归一化数据到统一的csv文件中 ###

提交历史信息分别来源于版本管理软件svn和perforce。数据导出的格式一个是xml,一个是文本。
在图表化之前，需要归一化提交历史信息。[CSV]是一个结构性清晰，而且被很多商业软件支持的文件格式。
因此,选择csv作为归一化数据的文本格式，每一项由(author,date,msg)构成。

#### svn to csv ####

python读取xml格式的文件相当容易。

```python
import  xml.etree.ElementTree as ET

def data_preprocess_for_svn():
    with open('log_svn.csv', 'w', encoding='utf-8') as f:
        f.write('author,date,msg\n')
        tree = ET.parse('log_svn.xml')
        root = tree.getroot()

        for child in root:
            author  = ''
            date    = ''
            msg     = ''
            for subchild in child:
                if subchild.tag == 'author':
                    author = subchild.text
                elif subchild.tag == 'date':
                    date = subchild.text
                elif subchild.tag == 'msg':
                    msg = subchild.text
                
            date_re = re.match(r'\d{4}-\d{2}-\d{2}', date)
            if msg:
                msg_re = re.match(r'.*】(.*)', msg)
            else:
                msg = None
            if msg_re and len(msg_re.group(1)) > 0:
                f.write(author+','+ ''.join(date_re.group(0).split('-'))+','+ msg_re.group(1)+'\n')
```

svn的*date*过于具体，这里只需要具体到哪一天即可。

```python
<date>2017-12-13T14:06:33.374887Z</date>
```

因此，做如下处理：

```python
date_re = re.match(r'\d{4}-\d{2}-\d{2}', date)
date_text = ''.join(date_re.group(0).split('-'))
```

#### p4v to csv ####

从log信息中提取出每个changelist到一个数组里。
```
# content: p4v 导出的log信息
change_list = content.split('Change')
# t[0]: author & date, t[1:]: msg
t = change.split('\n')
```

从changelist中提取author。author的信息是在email的@的前面的字符串。
```python
# Change 1 on 2017/12/14 by zhangsan@email.com
author = re.search(r' ([a-zA-Z0-9]+)@', t[0]) 
```

从changelist中提取date。
```python
# Change 1 on 2017/12/14 by zhangsan@email.com
m = re.search(r'(\d{4}/\d{2}/\d{2})', t[0])
```

从changelist中提取msg
```python
msg = ''.join(t[1:]).strip()
```

### 图表化提交历史数据 ###

提交次数的按月份分布如下图：
![my commits](/assets/images/work_experience/my_commit.jpg "my commits")

在两年里，提交了477次左右，占总提交的1.7%。
![commits](/assets/images/work_experience/commits.png "commits")



两年内的提交按功能的散列图如下：

![module time line](/assets/images/work_experience/time-line.png "module time line")

由图可以清晰的看出，早期是从postprocess开始的，那时跟一个大大再搞一个real-time composition，大大负责整体框架的构建，大大指导我在
框架里添砖加瓦，aa,dof，bokeh,lensflare,tone mappint, color grade, xdog等等相关的后期效果都倒腾了一番。期间，一个前辈丢了个shadow的锅给我们组，自己跑去专研地形渲染。
大大随手一丢，就到我手上了，接下来就一接就接到现在了。sm,vsm，msm,sdsm, height-based ao...。接了shadow,也接了pointlihgt's shadow。然后
就接了directional light， spot light。最后，就是decal。这个功能很早就提出需求了，也在现有的版本里凑出了功能，产品化时觉得耦合度过高，还有
当时还有其他的比较紧急的任务就耽搁了一段时间。接下来，就是把耦合度降低，单独分出来，功能提交了也验证。一入码农，随处bug。代码在[github]。

### References ###

* [Comma-separated values](https://en.wikipedia.org/wiki/Comma-separated_values)
* [python code - github](https://github.com/jamelouis/python-practice/blob/master/history/works.py)

[CSV]:https://en.wikipedia.org/wiki/Comma-separated_values
[github]:https://github.com/jamelouis/python-practice/blob/master/history/works.py