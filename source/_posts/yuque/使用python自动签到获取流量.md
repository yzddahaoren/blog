---
title: 使用python自动签到获取流量
urlname: pzgrf1
date: 2020-06-19 16:33:09 +0800
tags: []
categories: []
---

> 发现一个 0.5 毛一个月,而且还支持签到流量的机场,美滋滋
> 直接写个脚本每天捞点流量

## 环境

- Google 浏览器
- chromedriver
- python3

## 具体

### 服务器环境安装

python3

```shell
mkdir /usr/local/python3
wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz
tar -xvf Python-3.6.3.tgz
//编译
cd Python-3.6.3
./configure --prefix=/usr/local/python3Dir
make
make install
//报错
yum -y install gcc
//软连接
//进入/usr/local/python3Dir
ln -s /usr/local/python3Dir/python3 /usr/bin/python3
ln -s /usr/local/python3Dir/pip3 /usr/bin/pip3
```

安装完成
修改 pip3 的默认配置

```shell
pip3 install pip -U
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install selenium
```

### 安装浏览器环境

```shell
# 安装chrome-browser
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm --no-check-certificate
sudo yum install google-chrome-stable_current_x86_64.rpm
google-chrome --version //查看版本
// 安装chromedviver
//https://chromedriver.chromium.org/ 下载对应版本的二进制文件
mv chromedriver /usr/local/bin
```

### 搭建可用的代理接口

- 见前一篇文章 `搭建telegarm的代理`

### python 脚本

```python
#coding=utf-8
import time
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException

username = 'name'  # 账号
password = 'passwd'  # 密码
login_url = 'https://paoluz.com/auth/login'  # 登录URL
checkin_url = 'https://paoluz.com/user'  # 签到URL



chrome_options = webdriver.ChromeOptions()
#chrome_options.set_headless()
#chrome_options = Options()
chrome_options.add_argument('--headless') # 16年之后，chrome给出的解决办法，抢了PhantomJS饭碗
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--proxy-server=socks://39.96.81.114:10808')
driver = webdriver.Chrome(chrome_options = chrome_options)
driver.maximize_window()  # 最大化窗口
driver.get(login_url)  # 进入登录页面

try:
    driver.find_element_by_id('email').send_keys(username)  # 填充用户名和密码
    driver.find_element_by_id('password').send_keys(password)
    driver.find_element_by_xpath('.//button').click()  # 登录
    time.sleep(1.5)
    driver.get(checkin_url)
    try:  # 未签到
        driver.find_element_by_xpath("//*[@class='btn btn-icon icon-left btn-primary']").click()  # 签到
        time.sleep(5)
        print("签到成功")
    except NoSuchElementException:
        print("已签到")
except Exception as e:
    print(e)
    print("签到失败")

driver.quit()
```

修改 `name`  和 `passwd`  即可

```python
python3 文件名.py //即可正常签到
```

后面设定定时任务即可,即可每天签到获取流量
