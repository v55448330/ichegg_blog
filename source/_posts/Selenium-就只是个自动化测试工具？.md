---
title: Selenium 就只是个自动化测试工具？
date: 2016-11-26 17:58:30
tags:
categories: 开发
---
## 前言
最近项目中碰到一个比较 **奇葩** 的需求，记录下来先。某公司就不说了 - - 因为 **信息安全** 政策，内网任何数据 **严禁** 外传，但是测试机在 **外网** ，所以需要解决 **软件包发布** 到测试机的问题。

开发环境为 Android ，18 台测试手机，编译出的 APK 有两种渠道到达测试机
1. 通过 **授权** 的 USB 连接使用 adb 方式 **部署** 至手机
2. 通过 **手动上传** 包至公司的 **应用发布平台** ，使用手机打开链接下载
<!--more-->

两种方式都有几个共同的问题：
 1. **效率低下**
 2. **效率低下**
 3. **效率低下**

作为一个提倡 **敏捷** 的高大上企业，每天出包不计其数，尤其像本人这样的懒癌晚期 **DevOps** ，是坚决忍受不了的！

## 有啥招
分析下问题

渠道一

可以通过 **脚本** 方式实现自动化，但是随着测试机 **数量** 和 **种类** 的增加，apk 包 **产出量** 增加，无论从 **端口数量** 或者 **USB** 接口 性能、供电限制，都将导致 **性能** 直线下降甚至不可用

渠道二

看起来很美好，如果有 **API** ，借助 **发布平台** ，可以不需要考虑性能，存储等问题，集成 CI 后可以完美实现 **自动化** 发布，但是由于传输文件的 **安全政策** ，此处基本为妄想 ... 

## 没招了吗

基于以上分析，渠道一不具备 **扩展性** 并且存在 **性能风险** 基本 pass，那么渠道二看起来 **只要** 解决了 **API** 的问题就可以 **实现** 自动发布了。

其实换个思路，**目的** 是通过 Web 进行 **上传包** 操作，如果 API 无法提供，那么根据很久以前牛逼的 **模拟页面操作** 充 Q 币的玩法是不是也可以实现同样的结果？

第一反应，当年的玩法，基于 **.Net Framework** 的 **WebBrowser** 控件实现模拟操作，由于需要安装 Visual Stdio 放弃 ...

随即请出今天的主角 --> **[Selenium](www.seleniumhq.org)** ！

## 介绍下先

**什么是 Selenium**

Selenium automates browsers. That's it! What you do with that power is entirely up to you. Primarily, it is for automating web applications for testing purposes, but is certainly not limited to just that. Boring web-based administration tasks can (and should!) also be automated as well.

Selenium has the support of some of the largest browser vendors who have taken (or are taking) steps to make Selenium a native part of their browser. It is also the core technology in countless other browser automation tools, APIs and frameworks.

上面这串我写不出来的文字翻译过来大概就是，这是一个 **很牛 X 的 Web 自动化测试** 框架

**为什么选这玩意**

1. 有 Python 的扩展包，而且很小
2. 几乎可以完美的 **模拟用户行为** 操作 
3. 使用 WebDriver 的方式 **兼容绝大多数** 桌面、移动浏览器并且支持 PhantomJS
4. **创始人曾是 ThoughtWorks 的咨询师**

> 当然，重点是第四点

## 怎么搞

### WebDriver

Selenium 通过 [WebDriver](http://www.seleniumhq.org/projects/webdriver/) 控制浏览器实现操作。

WebDriver 分为两种，**真实浏览器** 以及 **伪浏览器**，大体如下：

**真实浏览器：**
- Firefox Driver
- Safari Driver
- IE Driver
- Chrome Driver
- Opera Driver

**伪浏览器**
- PhantomJS Driver
- HtmlUnit Driver

伪浏览器 **不提供** UI，速度相对较 **快** ，适合非 GUI 的 **功能** 测试。

真实浏览器可以 **真实模拟用户行为** 并实时展示，速度也相对较 **慢** 。

更多的 WebDriver 请参照官方文档。

### 安装

> 这里使用 Mac，WebDriver 使用 Chrome，其他组合大同小异，不做赘述

**安装 Selenium**

`pip install selenium`

**安装 Chrome Driver**

`brew install chromedriver`

> 更多版本请在 [这里](https://chromedriver.storage.googleapis.com/index.html) 下载
> 因为 Chrome 的 Driver 属于 **真实浏览器** ，所以还需要下载对应版本的 **Chrome**
> 使用 PhantomJS 则 **只需要** 下载 PhantomJS 即可，无需下载 Driver

### 示例

> 因为客户信息安全，所以这里 **只提供示例** 演示
> 示例通过实现一个简单的查询 IP 功能演示 **常用** 接口，可以举一反三完成绝大多数操作，更多的接口使用方法请参考官方文档

```
# -*- coding:utf-8 -*-
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
 
try:
    browser = webdriver.Chrome()
    # 实例化 ChromeDriver ，成功后会自动打开 Chrome 浏览器
    
    # chromeDriverDir = '/usr/bin/google-chrome'
    # browser = webdriver.Chrome(executable_path=chromeDriverDir)
    # 通过指定路径方式实例化 ChromeDriver

    browser.maximize_window()       # 最大化窗口
    browser.set_page_load_timeout(20)       # 设置页面超时时间
    
    url = "http://www.ip.cn"
    browser.get(url)
    browser.implicitly_wait(5)
    # 等待页面加载完成，最长 5 秒（此处页面内刷新无效）

    browser.find_element_by_xpath("//div/form/input[@name='ip'][@type='text']").send_keys('8.8.8.8')
    # 通过寻找指定结构中 name=ip 并且 type=text 的 input 对象并输入 8.8.8.8

    browser.find_element_by_xpath("//div/form/input[@type='submit'][@id='s']").click()
    # 通过寻找指定结构中 type=submit 并且 id=s 的 input 对象并单击它

    WebDriverWait(browser, 5).until(lambda brow: "8.8.8.8" in brow.title)
    # 等待浏览器返回，最长 5 秒，直到检测到页面标题中有 8.8.8.8 结束
    
    content = browser.find_elements_by_xpath("//div[@id='result']/div/p/code")
    # 获取指定结构下所有 code 对象

    print "查询成功！结果：" + str(content[1].text)
    # 取下标为 1 的对象文本输出

    # browser.quit()    # 关闭浏览器
     
except Exception as ex:
    print("error msg: " + str(ex))
```

## 参考文档

[https://realpython.com/blog/python/headless-selenium-testing-with-python-and-phantomjs/](https://realpython.com/blog/python/headless-selenium-testing-with-python-and-phantomjs/)
[http://www.seleniumhq.org/docs/](http://www.seleniumhq.org/docs/)
[http://phantomjs.org/documentation/](http://phantomjs.org/documentation/)