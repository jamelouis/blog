---
layout: post
title:  "Hackerrank Challenges Overview"
date:   2017-12-16 22:03:31 +0800
categories: python
---

最近想尝试用编程的方式来重温一些数学的知识，如微积分，以免以后“书到用时方恨少”。经过一番的Google, [Hackerrank]的[Mathematics]就是我想要的。
凡事预则立不预则废，得好好规划规划。这里，就来分析分析大概需要多长时间来完成这个小任务。

大致的思路如下：
1. 从[Hackerrank]上抓起所有challenges，以csv格式分别存储subdomains的challenges.
2. 先在execl表格，简单的测试challenges的柱状图和饼状图，接着用python脚本化
3. 对Challenges的难度系数估计相应的时间，估算出大致需要的时间

## 爬虫：抓起[mathematics]的所有challenges

爬虫的过程如下：
1. 用python的request库，获取网页源文件
2. 用python的BeautifulSoup和re库，来提起链接和challenge信息
3. 用python的io库，存储提起的信息

1,3相对比较简单，可以参考[Web Scraping with Python - Book],需要注意的事，[hackerrank]会检测request的headers信息，因此不能使用默认的headers信息。
反爬虫的技术也越来越成熟，为了不被封ip, 提起的过程是先将1爬起的结果存成文件，然后读取文件来提起。

### 信息提起

[mathematics]下的**Subdomains**有： Fundamentals, Number Theory, Combinatorics..., 每个subdomain根据challenges的个数不同，会用不同的页数来显示。
因此，需要先提起所有*Subdomains*的链接，然后提起所有*Subdomains*每个分页上的所有Challenges.

#### Subdomains提起
Google浏览器调试分析得出：

![Subdomains](/assets/images/hackerrank/subdomains.jpg "subdomains")

```python
def get_subdomain(html):
    soup = bs(html,'html.parser')
    return [(li.a['data-attr1'], li.a['href']) for li in soup.findAll('li', {'class':'chapter-item'})]
```
#### Challenges提起

Google浏览器调试分析得出：
![Challenges](/assets/images/hackerrank/challenge.jpg "challenge")
![pages](/assets/images/hackerrank/pages.jpg "pages")

```python
def get_problems(html):
    soup = bs(html,'html.parser')
   
    challenges_div = soup.findAll('div', { 'class' : 'challenges-list-view'})

    page_list = []
    problem_list = []

    for challenge_block in challenges_div:
        title = challenge_block.h4.get_text()
        #print(challenge_block.footer.get_text())
        difficulty = re.search(r'Difficulty: (Easy|Medium|Hard|Advanced)', challenge_block.footer.get_text())
        if difficulty:
            print(title, difficulty.group(1))
            problem_list.append((title, difficulty.group(1)))

    page_items = soup.findAll('li', { 'class' : 'page-item' } )

    for page in page_items:
        print(page.a['href'])
        page_list.append(page.a['href'])

    return page_list, problem_list
```

## 图表化

csv文件一般都被各个表格软件兼容。用excel软件打开，简单操作生成如下的bar图，pie图。

![excel bar pie图](/assets/images/hackerrank/excel-bar-pie.jpg "Excel bar pie图")

由图可知，Fundamental下有20道Easy难度，3道Hard难度，9道Medium难度。各占63%,9%,28%。

先前，各个不同的**Subdomains**各自存在各自的csv文件中，因此可以用python脚本化图表化过程。

![fundamentals](/assets/images/hackerrank/fundamentals.png "Efundamentals")

## 规划

由图表化中得，[mathematics]的challenges概览如下图：
![mathematic](/assets/images/hackerrank/mathematic.png "mathematic")

Easy难度的有45, Medium难度的有102，Hard难度的有80，Advanced难度的有29.

假设，Easy难度的耗时10min,Medium难度耗时30min, Hard难度耗时120min, Advanced难度耗时240min,
则需要 45 * 10 + 102 * 30 + 80 * 120 + 29 * 240 = 20070min，约334h.
如果每天能花4个小时，则需要83 days.

## 小结

* [Web Scraping with Python - Book] 是本不错的爬虫书籍
* [Python Documentation] 需要经常浏览
* [Source Code - github]

## References

* [Web Scraping with Python - Book](https://www.amazon.cn/dp/1491910291/ref=sr_1_2?ie=UTF8&qid=1513504275&sr=8-2&keywords=Web+Scraping+with+Python)
* [Python Documentation](https://docs.python.org/3/)

[Web Scraping with Python - Book]: https://www.amazon.cn/dp/1491910291/ref=sr_1_2?ie=UTF8&qid=1513504275&sr=8-2&keywords=Web+Scraping+with+Python
[Hackerrank]:https://www.hackerrank.com
[Mathematics]:https://www.hackerrank.com/domains/mathematics/fundamentals
[Python Documentation]:https://docs.python.org/3/
[Source Code - github]:https://github.com/jamelouis/python-practice/tree/master/hackerrank