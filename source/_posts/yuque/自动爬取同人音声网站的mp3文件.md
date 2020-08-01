---
title: 自动爬取同人音声网站的mp3文件
urlname: gv3srn
date: 2020-06-11 08:29:07 +0800
tags: []
categories: []
---

## ![kak.PNG](https://cdn.nlark.com/yuque/0/2020/png/1550831/1591836230716-307fd330-c1d8-48cf-909d-61018a2a861c.png#align=left&display=inline&height=534&margin=%5Bobject%20Object%5D&name=kak.PNG&originHeight=534&originWidth=1755&size=139186&status=done&style=none&width=1755)

## 介绍

最近发现了一个外国人搭建的同人音声网站 [https://japaneseasmr.com](https://japaneseasmr.com) ,admin 每天更新,遂爬之
但是还是不要太过分,看更新的频率应该不是脚本更新.

## 环境和工具

- 使用 IDM 破解版,因为直接就有方便
- v2ray 代理,你懂的原因

#### 前期准备

- IDM 设置代理下载(服务器在国外)
- v2ray 开启 http 代理

### 脚本编写

#### 实现功能

- 下载 mp3 文件
- 下载封面图片
- 去 dlsite 找文件详细信息

#### 开始

```python
import requests
from lxml import etree
from subprocess import call
import eyed3
import time
import os

headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36'
}
base_url = 'https://japaneseasmr.com/tag/'
proxies = {"http": "http://127.0.0.1:10809"}


def main(tag):
    tag_true = str(tag)
    url = base_url + '{}'.format(tag_true)
    resqonse = requests.get(url,headers=headers,proxies=proxies)
    html = resqonse.text
    link_html = etree.HTML(html)
    page_num = link_html.xpath('//div[@class="nav-links"]/a[@class="page-numbers"]')[1]
    for i in int(page_num)+1:
        nuw_url =  url + '/page/' + '{}'.format(i+1)
        new_response = requests.get(url=nuw_url,headers=headers,proxies=proxies)
        new_html = etree.HTML(new_response)
    # print(link_html)
        return new_html
        urls = get_urls(new_html)
        get_music(urls)

def get_urls(new_html):
    urls = new_html.xpath('//div[@class="entry-preview-wrapper clearfix"]/h2/a/@href')
    # print(urls)
    return urls

def get_music(urls):
    for url in urls:
        url_response = requests.get(url,headers=headers,proxies=proxies)
        url_html = url_response.text
        url_link = etree.HTML(url_html)
        download_link = url_link.xpath('//div[@class="audio_main"]/audio/source[@id="backup_a"]/@src')
        # path(download_link)
        download(download_link)
        time.sleep(1)
        # print(download_link)


def download(download_link):
    IDM = r'F:\Program Files\xiazai\Internet Download Manager\IDMan.exe'
    out_name = download_link[0].split('/')[-1]
    dir_name = out_name.split('.')[0]
    rj = dir_name[2:]
    rji = int(rj)
    rjh = ((rji//1000)+1)*1000
    save_path = r'G:\1A-MUSIC\NEW\thing\{}'.format(dir_name)
    img_path = 'https://img.dlsite.jp/modpub/images2/work/doujin/'+'RJ'+'{}'.format(rjh) +'/'+ dir_name + '_img_main.jpg'
    # img_path = 'https://pic.weeabo0.xyz/' + dir_name + '_img_main.jpg'
    img_name = dir_name + '.jpg'
    print('-'*30+'正在保存{}'.format(dir_name)+'-'*30)
    call([IDM, '/d' ,download_link, '/p' ,save_path, '/f' ,out_name, '/n' , '/a'])
    time.sleep(0.1)
    call([IDM, '/d' ,img_path, '/p' ,save_path, '/f' ,img_name, '/n' , '/a'])
    # print('-'*30+'开始合并文件'+'-'*30)
    # mp3(save_path,out_name,img_name)
    # print('-'*30+'合并文件完成'+'-'*30)
    text_hanshu = get_dlsite(dir_name,save_path)

def get_dlsite(dir_name,save_path):
    dlsite_link = 'https://www.dlsite.com/maniax/work/=/product_id/{}.html'.format(dir_name)
    dlsite_response = requests.get(url=dlsite_link,headers=headers,proxies=proxies)
    dlsite_html = etree.HTML(dlsite_response.text)
    save_text_name = dlsite_html.xpath('//div[@class="base_title_br clearfix"]/h1/a/text()')[0]
    message = dlsite_html.xpath('//div[@id="work_right_inner"]')[0]
    text_name = save_path + r'\{}'.format(dir_name) + '.txt'
    f = open(text_name,'a')
    f.write(save_text_name +'\n')
    f.write(message + '\n')
    f.close

if __name__ == '__main__':
    tag = input('输入标签:\t')
    main(tag)
    # main('ritsuka-mizutani')
```

#### 说明

因为懒得改了,说明下吧:

- 自己的 IDM.exe 文件位置为 `F:\Program Files\xiazai\Internet Download Manager\IDMan.exe`
- 自己的文件下载目录为 `G:\1A-MUSIC\NEW\thing\{}` 其中{}内为每一个的 RJ 号
- 其实完成后内含的 txt 里面包含 RJ 名字,但是可以搭配这个 [https://github.com/Watanuki-Kimihiro/Doujin_Voice_Renamer](https://github.com/Watanuki-Kimihiro/Doujin_Voice_Renamer) 使用,方便管理.

#### 后续

- 给 mp3 文件添加封面,美观一点
- 百度云自动上传和分享,关键是现在好用的百度云命令行没有了, `baidupcs-go`  多久没更新了

### 示意图

![kak.PNG](https://cdn.nlark.com/yuque/0/2020/png/1550831/1591836213131-eb1073be-6b9a-419b-9273-4b6940768ca4.png#align=left&display=inline&height=534&margin=%5Bobject%20Object%5D&name=kak.PNG&originHeight=534&originWidth=1755&size=139186&status=done&style=none&width=1755)
