---
title: telegram使用代理
urlname: pw5fuf
date: 2020-06-15 19:57:37 +0800
tags: []
categories: []
---

## ![566189dd2240950b7a8960535121cfb3.png](https://cdn.nlark.com/yuque/0/2020/png/1550831/1592222691120-95634c6e-92eb-47e2-ba1c-aa74bd738a01.png#align=left&display=inline&height=175&margin=%5Bobject%20Object%5D&name=566189dd2240950b7a8960535121cfb3.png&originHeight=175&originWidth=404&size=9745&status=done&style=none&width=404)

## 具体方式

- v2ray 转 socks5 代理 (有服务器即可,国内的也可)
- MTDPROXY telegram 官方的代理 (需要有主权的国外服务器)

## 安装

### 第一种

脚本在安装 MTProxy
[mtproxy_go.sh](https://www.yuque.com/attachments/yuque/0/2020/sh/1550831/1592222611562-4bb2d085-de14-4f74-b346-797c890a2954.sh?_lake_card=%7B%22uid%22%3A%221592222612396-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Fsh%2F1550831%2F1592222611562-4bb2d085-de14-4f74-b346-797c890a2954.sh%22%2C%22name%22%3A%22mtproxy_go.sh%22%2C%22size%22%3A28057%2C%22type%22%3A%22text%2Fx-sh%22%2C%22ext%22%3A%22sh%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22B3Sbf%22%2C%22card%22%3A%22file%22%7D)

### 第二种

利用 v2ray 容器当服务端

1. vmess2json.py

[vemss2json.py](https://www.yuque.com/attachments/yuque/0/2020/txt/1550831/1592222498021-9856ae6d-1e7b-4d64-80ed-b085b1954175.txt?_lake_card=%7B%22uid%22%3A%221592222498691-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Ftxt%2F1550831%2F1592222498021-9856ae6d-1e7b-4d64-80ed-b085b1954175.txt%22%2C%22name%22%3A%22vemss2json.py%22%2C%22size%22%3A20877%2C%22type%22%3A%22text%2Fplain%22%2C%22ext%22%3A%22txt%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22xCjvL%22%2C%22card%22%3A%22file%22%7D)   保存成 config.josn 文件 注意改监听地址为 0.0.0.0

2. 运行命令即可

`docker run -d -v $PWD:/etc/v2ray -p 10808:1080 --name v2 v2ray/official`

3. sock5 加密 修改如下字段即可

```json
"settings": {
                "auth": "password",
                "accounts": [
                {
                    "user": "yzddhr",
                    "pass": "passwd"
                }
                ],
```
