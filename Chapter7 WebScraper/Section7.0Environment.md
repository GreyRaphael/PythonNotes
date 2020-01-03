# Spider Environment

- [Spider Environment](#spider-environment)
  - [Fiddler](#fiddler)
  - [Outline](#outline)
  - [Trick](#trick)

## Fiddler

for android platform
- 安装Fiddler
- HTTPS/Capture HTTPS CONNECTS/Decrypt HTTPS traffic; Ignore server certificate errors
- Connections/Allow remote computers to connect, port=8888
- PC开热点
- Android连接热点，并设置wifi代理: `192.168.137.1:8888`
- Android 7之前的手机只需要安装`FiddlerRoot.cer`证书即可；Android7需要root到系统证书(十分麻烦)

也可以通过雷电模拟器+Fiddler实现抓包

## Outline

第一部分 环境篇
1. Python3+Pip环境配置
1. MongoDB环境配置
1. Redis环境配置
1. MySQL环境配置
1. Python多版本共存配置
1. Python爬虫常用库的安装

第二部分 基础篇
1. 爬虫基本原理
1. Urllib库基本使用
1. Requests库基本使用
1. 正则表达式基础
1. BeautifulSoup详解
1. PyQuery详解
1. Selenium详解

第三部分 实战篇
1. 使用Requests+正则表达式爬取猫眼电影
1. 分析Ajax请求并抓取今日头条街拍美图
1. 使用Selenium模拟浏览器抓取淘宝商品美食信息
1. 使用Redis+Flask维护动态代理池
1. 使用代理处理反爬抓取微信文章
1. 使用Redis+Flask维护动态Cookies池

第四部分 框架篇 
1. PySpider框架基本使用及抓取TripAdvisor实战
1. PySpider架构概述及用法详解
1. Scrapy框架的安装
1. Scrapy框架基本使用
1. Scrapy命令行详解
1. Scrapy中选择器的用法
1. Scrapy中Spiders的用法
1. Scrapy中Item Pipeline的用法
1. Scrapy中Download Middleware的用法
1. Scrapy爬取知乎用户信息实战
1. Scrapy+Cookies池抓取新浪微博
1. Scrapy+Tushare爬取微博股票数据

第五部分 分布式篇
1. Scrapy分布式原理及Scrapy-Redis源码解析
1. Scrapy分布式架构搭建抓取知乎
1. Scrapy分布式的部署详解

## Trick

动态网页: html+js加载
- 分析ajax请求
- selenium+browser
- [Splash](https://github.com/scrapinghub/splash)
- [PyV8 and Ghost.py](https://github.com/jeanphix/Ghost.py)