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
3. find_element 太麻烦了，要不停的对xpath
4. 还要点击链接 switch windows 来获取招聘具体内容
5. 查询框clear 清除不掉， 简单办法直接ctrl a 全选再修改
6. 页面切换还要有点延时，不然找不到element

反爬程度   boss直聘 〉51job 〉智联招聘 〉 猎聘

单线程跑有点慢，不用ip代理一会就被检测出是爬虫，懒得弄了。

1. boss直聘需要点击按钮，按顺序识别图像，不加sleep很快就被识别出来
2. 51job 是滑动窗口检测，很有可能被判定为webdriver，要想办法避免
3. 智联只是登陆要用手机号，这个要搞服务中转
4. 猎聘没碰到啥，只是搜索的职位很奇怪

招聘网站不提供职位发布时间太蛋疼。

代码直接看链接： [spider](https://github.com/song-py/spider)
