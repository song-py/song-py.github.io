+++
title = '爬招聘网站'
date = 2024-04-30T20:03:03+08:00
ags = ['爬虫', 'selenium']
draft = false
+++

个人需求需要看下招聘网站，手动翻太麻烦了，简单写个程序爬一下数据吧。

这个不是傻瓜式爬虫教程，只是记录下过程。

python要么直接用requests库或更底层的urlib3，要么用selenium 库模拟web操作，人比较懒直接用了selenium库

遇到的问题记录一下：

1. chromedriver 要版本一致，路径要配置好
2. option 增加参数避免webdriver 检测
3. 滑动窗口检测，很有可能被判定为webdriver，想办法避免
4. find_element 太麻烦了，要不停的对xpath
5. 还要点击链接 switch windows 来获取招聘具体内容
6. 查询框clear 清除不掉， 简单办法直接ctrl a 全选再修改
7. 页面切换还要有点延时，不然找不到element
8. 单线程跑有点慢，不用ip代理一会就被检测出是爬虫，懒得弄的

适配了51job， 再弄其他两家网站试试

代码直接看链接： [spider](https://github.com/song-py/spider)
