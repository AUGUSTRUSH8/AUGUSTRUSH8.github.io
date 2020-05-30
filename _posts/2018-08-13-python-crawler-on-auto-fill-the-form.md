---
layout: post
title: 'Python+Selenium实现自动报税'
tags: [code]
---

> 缘起：开发当中有这么一个需求---根据后台提供的财务数据（多半是一些公司的月度或季度年度财务报告、资产负债表等），自动模拟登录当地报税网站，自动完成form表单的填写和提交，然后需要跟踪报税进度。

**分析**：这其中的工作，大体可拆分为几个阶段

- 第一，相关公司报税负责人的登录信息，可解析对应存储文件获取
- 第二，根据报税人的账号密码登录报税网站
- 第三，根据财务报表逐个解析并切分页面进行填写，最终提交

**难点**：其中难点也是比较多的

- 第一，报税网站采取的是RSA非对称加密方式认证用户登录
- 第二，网站登陆页有图片验证码
- 第三，页面采取的技术比较老，使用的是iframe嵌入的方式
- 第四，页面数据多采用异步加载的方式，并不是一开始就把某些数据渲染到页面的。（这当然咯）
- 第五，页面本身很多表单项并不都是靠填写进去的，当填写完某一项之后，其他的某些项可以自行应用js进行计算

#### 尝试

由于以前并没有接触过这类业务，使用的技术只有一个大概的方向，并不明朗。所以经过查资料看相似项目源码大体知道可以使用爬虫的方式解决

- 先使用httpclient还有jsoup的原生方式进行请求解析，其中登录进去获取token费了不少功夫，构造加密请求串，获取验证码并智能识别字符。。。最后终于如愿以偿
- 然而最后还是卡在了iframe标签上，总是无法加载iframe当中的内容，StackOverflow等各个地方都有去问询了解，然而还是卡住了，其实是可以解决的。但我转念再去考虑这个业务特点的时候突然发现selenium驱动浏览器不失为一个好办法！

所以诞生了下面的解决方案，Python+Selenium+Webdriver

```python 
'''
Created on 2018年10月30日

@author: AUGUSTRUSH
'''
'''
dzswj.cq-l-tax.gov.cn
12366.cqsw.gov.cn
以上两个网址都可以登进系统

尝试使用OCR的方式识别验证码，但命中识别率较低，因此放弃使用
可将验证码发送给点击一键纳税的客户，让用户输入结果
为避免多个用户同时发起请求，可队列限制请求频率
'''
#前提：需要pip后者conda安装selenium包
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import csv
#登录函数
def Login(driver):
    driver.get("http://12366.cqsw.gov.cn:7004/ssoserver/login")
    #延时一秒
    time.sleep(1)
    #截取当前登录页面图片并保存
    driver.save_screenshot("../saved_images/报税网首页.png")
    #获取当前需要登录的用户名密码
    (username, password), = readCSVFile().items()
    #输入账号
    driver.find_element_by_id("username").send_keys(username)
    #输入密码
    driver.find_element_by_name("password").send_keys(password)
    #保存验证码的图片
    driver.save_screenshot("../saved_images/验证码.png")
    #阻塞输入验证码
    check_code = input("请输入验证码:")
    print(r"验证码是多少:%s" % check_code)
    #向登录页面输入验证码信息
    driver.find_element_by_id("captcha").send_keys(check_code)
    #点击登录按钮
    driver.find_element_by_xpath("//*[@id=\"zrrLoginBtn\"]").click()
    #休眠一下等待登录成功
    time.sleep(3)
    #保存登录成功的快照
    driver.save_screenshot("../saved_images/登录成功.png")
    #消除弹窗提示 此处需注意，该网站会不定期的在初次登录时候弹出通知信息框
    #因此这里的弹窗信息是不确定的，但每次都是调用相同的弹窗组件提示显示，
    #所以以下执行两次关闭弹窗点击
    driver.find_element_by_id("closeWid").click()
    driver.find_element_by_id("closeWid").click()
    #截取点击完弹窗按钮后的页面
    driver.save_screenshot("../saved_images/消除弹窗主页面.png")
    #销毁无头浏览器进程--重要，否则会一直阻塞
    driver.quit()
#点击"申请纳税"按钮
def clickPayTaxButton(driver):
    #点击申报纳税a标签
    driver.find_element_by_xpath("//*[@id=\"left-contain\"]/ul[1]/li[2]/ul/li[4]/a").click()
#begin----增值税纳税函数模块起始
#点击"增值税小规模纳税人申报"a标签
#此处会跨域请求获取资源，并展现在当前页面当中，当前window_handle
def clickThroughVATaTag():
    #点击增值税纳税a标签
    driver.find_element_by_xpath("//*[@id=\"left-contain\"]/ul[1]/li[2]/ul/li[4]/ul/li[1]/a").click()
    #保存截图
    driver.save_screenshot("../saved_images/增值税.png")
    #保存当前页面的html到本地
    with open("../saved_htmlFile/cqTax.html","w",encoding="utf-8") as f:
        f.write(driver.page_source)
    #切换到iframe里面去
    driver.switch_to.frame("mainframe")
    #点击填表申报
    driver.find_element_by_xpath("/html/body/div[2]/div[2]/ul[1]/li[5]/a").click();
    #点击后等待5秒
    time.sleep(5)
    # 拿到所有的窗口
    allHandles = driver.window_handles
    routeWindowHandle = 0
    routeTitle = 'None'
    for handle in allHandles:
        if handle!=driver.current_window_handle:
            #切换到新开页面
            driver.switch_to_window(handle)
            routeWindowHandle=driver.current_window_handle
            routeTitle=driver.title
            break
    time.sleep(6)
    print(f"4. switch to routeWindow, routeWindowHandle = {routeWindowHandle},routeTitle = {routeTitle}, currentHandle = {driver.current_window_handle}, currentTitle = {driver.title}")
    #截取当前页面
    driver.save_screenshot("../saved_images/点击填表申报以后.png")
#end----增值税纳税函数模块结束
def readCSVFile():
    # 读取csv至字典
    csvFile = open("../usernameAndPassword/userInfo.csv", "r")
    reader = csv.reader(csvFile)
    # 建立空字典
    result = {}
    for item in reader:
        # 忽略第一行
        if reader.line_num == 1:
            continue
        result[item[0]] = item[1]
    csvFile.close()
    return result
def main():
    #初始化浏览器
    chrome_options = Options()
    chrome_options.add_argument('window-size=1366x768') #指定浏览器分辨率
    chrome_options.add_argument('--disable-gpu') #谷歌文档提到需要加上这个属性来规避bug
    chrome_options.add_argument('--hide-scrollbars') #隐藏滚动条, 应对一些特殊页面
    chrome_options.add_argument('--headless') #浏览器不提供可视化页面. linux下如果系统不支持可视化不加这条会启动失败
    #初始化一个局部对象driver
    driver = webdriver.Chrome(executable_path="../chrome_driverFile/chromedriver_win32/chromedriver.exe",chrome_options=chrome_options)
    return driver
if __name__ == "__main__":
    #获得初始化了参数的driver对象，它一旦初始化就全局使用
    driver=main()
    #执行登录操作
    Login(driver)
    print("登录结束")
    #首页打开后，默认会展开"办税服务"下拉菜单，然后需要
    #点击"申请纳税"子菜单，否则会报invisible异常（所见即所得）
    clickPayTaxButton(driver)

```

这其中涉及到挺多知识点的，需要有个整体的把控。