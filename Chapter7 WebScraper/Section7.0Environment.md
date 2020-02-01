# Spider Environment

- [Spider Environment](#spider-environment)
  - [Introduction](#introduction)
  - [Fiddler](#fiddler)
  - [Outline](#outline)
  - [Security Level](#security-level)

## Introduction

爬虫: **请求**网站并**提取**数据的**自动化**程序

爬虫基本流程:
1. 发起请求: 通过HTTP库向目标站点发起请求，即发送一个Request，请求可以包含额外的headers等信息，等待服务器响应。
1. 获取响应内容: 如果服务器能正常响应，会得到一个Response，Response的内容便是所要获取的页面内容，类型可能有HTML，Json字符串，二进制数据（如图片视频）等类型。
1. 解析内容: 得到的内容可能是HTML，可以用正则表达式、网页解析库进行解析。可能是Json，可以直接转为Json对象解析，可能是二进制数据，可以做保存或者进一步的处理。
1. 保存数据: 保存形式多样，可以存为文本，也可以保存至数据库，或者保存特定格式的文件。

抓取静态页面:
- urllib2, urllib.request
- requests
- selenium
- scrapy

抓取动态页面:
- scrapy-splash
- selenium
- requests访问json

加快下载速度:
- 协程
- 多线程
- 多进程
- 分布式

反反爬虫:
- UserAgent
- Proxy
- disable cookie
- selenium, scrapy-splash
- 验证码: tesseract训练数据识别; tensorflow识别; 

爬虫数据解析方式:
- 直接解析
- json解析: `json.loads()`
- re
- xpath
- beautifulsoup
- pyquery
- jsonpath

保存数据:
- json, txt, xml, csv, xls
- mysql, mongodb, redis
- scrapy mysql, mongodb 异步保存

数据可视化:
- pyecharts
- matplotlib
- seaborn
- wordcloud

数据视角
- 爬虫: 抓取数据
- 运维: 部署、监控、分布式爬虫、分布式作业
- web: 管理数据、权限、展示数据
- 数据分析: 数据加工生成报告; AI

js渲染界面解决方式:
- 分析ajax请求
- selenium+browser
- [Splash](https://github.com/scrapinghub/splash)
- [PyV8 and Ghost.py](https://github.com/jeanphix/Ghost.py)

数据保存方式:
- 文本: plain text, json, xml
- 关系数据库: mysql, oracle, sql server
- 非关系数据库: mongodb, redis
- 二级制文件: 图片、音视频

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

## Security Level

- cookie: 可以重新连接
- session：重连需要验证
- token: 根据用户名、密码、ip判断是否是客户端，防止网络欺骗

不许登录site:
- 静态html: 
  - urllib
  - requests
- 动态页面(js, ajax):
  - requests抓json
  - selenium
  - scrapy-splash渲染js得到最终的html

需要登录的site:
- 通用办法selenium登录得到cookie+requests
- requests的session post登录(适用于很少的网站)，也可获得cookie
- requests使用浏览器的cookie来访问
- 研究网站的js加密过程，将加密的数据post来登录

带token的网站:
- 服务器提供token给客户端，客户端需要利用token和js来加密数据，然后客户端将加密的数据post来进行登录
- 服务器会经常修改token，导致post难度加大

example: token

本地数据是hello, token是1，js是加密方式(字母+1)，那么得到ifmmp;

如果服务器提供的token变成2，甚至js改变，post会变得异常困难