---
title: 自动部署语雀的文章到GitHub的hexo博客
urlname: rx5tq1
date: 2020-06-06 07:54:31 +0800
tags: []
categories: []
---

技术   教程

## 环境以及工具

- GitHub
- Travis-ci
- hexo + node
- 腾讯云服务器

## 不足以及优点

### 优点：

- 部署方便。免费
- 之后的管理就不用服务器了
- 比原生的管理后端好用太多比如什么**hexo-admin,hexo-editor**.

### 缺点:

- 图片无法外链

### 其他说明：

- 使用的是自己的管理仓库
- 需要配置 heox 的自动同步
- 需要把公钥写入 GitHub，**注意：设置密钥的权限是 read/write，即必须有写入的权限。**

## 搭建步骤

### 安装 ruby 和 gem

```shell
curl -L https://get.rvm.io | bash -s stable //安装ruby管理rvm
source /etc/profile.d/rvm.sh
echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > /usr/local/rvm/user/db
rvm install 2.3 //以安装2.3为例
gem install travis  //安装travis
```

### 配置私钥

1. 生成私钥

```shell
ssh-keygen -t rsa -C "youremail@example.com"   //生成密钥对
```

这个里面的 pub 公钥和 GitHub 的里面的公钥是一个，注意后面的配置

2. 加密私 🔑

```shell
gem install travis  //安装工具
```

3. 命令行登陆

```shell
travis login --github-token  XXX   //这个token在github账户的设置的里面生成，注意repo的权限全部打开
```

4. 新建一个文件夹复制文件

```shell
mkdir .travis
cd .travis
cp ~/.ssh/id_rsa .
```

5. 加密私钥

```shell
 travis encrypt-file id_rsa --add
```

会生成一个命令行的命令，记录下来，修改：

```shell
openssl aes-256-cbc -K $encrypted_f217180e22ee_key -iv $encrypted_f217180e22ee_iv -in {.travis}/id_rsa.enc -out {.travis}/id_rsa -d
```

其中括号里面的内容是自己加的，就是加了一个文件夹路径。

6. ssh 配置

在.travis 文件夹新建文件**ssh_config**

```shell
Host github.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```

7. travis 的网站配置
   1. travis 开启自己的 heox 的博客管理仓库 （即是博客的本体）
   1. 打开**general**的**build pushed branches**以及**build pushed pull requests** 打开
   1. 打开**auto cancellation**的**auto cancel branch builds **和 **auto cancel request builds**
8. 新建.travis.yml 文件

注意文件在 hexo 博客的根目录

```shell
language: node_js
node_js:
  - lts/*

cache:
  directories:
    - node_modules

before_install:
  - export TZ='Asia/Shanghai'
  - openssl aes-256-cbc -K $encrypted_f21718xxxxee_key -iv $encrypted_f21xxxx2ee_iv  -in .travis/id_rsa.enc -out .travis/id_rsa -d
  - cp .travis/id_rsa  ~/.ssh/id_rsa
  - chmod 600 ~/.ssh/id_rsa
  - eval $(ssh-agent)
  - ssh-add ~/.ssh/id_rsa
  - cp .travis/ssh_config ~/.ssh/config
  - git config --global user.name "yzxxhr"
  - git config --global user.email yzdxxxoren@126.com

install:
  - npm install hexo-cli -g
  - npm install

script:
  - npm run deploy

branches:
  only:
    - master
```

9. 配置云雀插件

```shell
npm install yuque-hexo //安装这个同步插件
```

### 配置 package.json 文件

```shell
"yuqueConfig": {
    "postPath": "source/_posts/yuque",
    "cachePath": "yuque.json",
    "mdNameFormat": "title",
    "adapter": "hexo",
    "concurrency": 5,
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "自己的用户名",
    "repo": "知识库的ID",
    "onlyPublished": false,
    "token": "自己的oath的id"
  },
  "scripts": {
    "clean": "hexo clean",
    "clean:yuque": "yuque-hexo clean",
    "deploy": "npm run sync && hexo deploy",
    "publish": "npm run clean && npm run deploy",
    "dev": "hexo s",
    "sync": "yuque-hexo sync",
    "reset": "npm run clean:yuque && npm run sync"
  },
```

在写入的时候注意某些逗号，并且**token**的获取在语雀的知识库的**oath**的获取，注意权限包含**文档内容获取**和**知识库获取**。

### 腾讯云的配置

登陆腾讯云的**云函数**,新建一个函数,函数的内容如下:

```shell
<?php
function main_handler($event, $context) {
    // 解析语雀post的数据
    $update_title = '';
    if($event->body){
        $yuque_data= json_decode($event->body);
        $update_title .= $yuque_data->data->title;
    }
    // default params
    $repos = 'xxxx';  // 你的仓库id 或 slug
    $token = 'xxxxxx'; // 你的登录token
    $message = date("Y/m/d").':yuque update:'.$update_title;
    $branch = 'master';
    // post params
    $queryString = $event->queryString;
    $q_token = $queryString->token ? $queryString->token : $token;
    $q_repos = $queryString->repos ? $queryString->repos : $repos;
    $q_message = $queryString->message ? $queryString->message : $message;
    $q_branch = $queryString->branch ? $queryString->branch : 'master';
    echo($q_token);
    echo('===');
    echo ($q_repos);
    echo ('===');
    echo ($q_message);
    echo ('===');
    echo ($q_branch);
    echo ('===');
    //request travis ci
    $res_info = triggerTravisCI($q_repos, $q_token, $q_message, $q_branch);

    $res_code = 0;
    $res_message = '未知';
    if($res_info['http_code']){
        $res_code = $res_info['http_code'];
        switch($res_info['http_code']){
            case 200:
            case 202:
                $res_message = 'success';
            break;
            default:
                $res_message = 'faild';
            break;
        }
    }
    $res = array(
        'status'=>$res_code,
        'message'=>$res_message
    );
    return $res;
}

/*
* @description  travis api , trigger a build
* @param $repos string 仓库ID、slug
* @param $token string 登录验证token
* @param $message string 触发信息
* @param $branch string 分支
* @return $info array 回包信息
*/
function triggerTravisCI ($repos, $token, $message='yuque update', $branch='master') {
    //初始化
    $curl = curl_init();
    //设置抓取的url
    curl_setopt($curl, CURLOPT_URL, 'https://api.travis-ci.org/repo/'.$repos.'/requests');
    //设置获取的信息以文件流的形式返回，而不是直接输出。
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    //设置post方式提交
    curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "POST");
    //设置post数据
    $post_data = json_encode(array(
        "request"=> array(
            "message"=>$message,
            "branch"=>$branch
        )
    ));
    $header = array(
      'Content-Type: application/json',
      'Travis-API-Version: 3',
      'Authorization:token '.$token,
      'Content-Length:' . strlen($post_data)
    );
    curl_setopt($curl, CURLOPT_HTTPHEADER, $header);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $post_data);
    //执行命令
    $data = curl_exec($curl);
    $info = curl_getinfo($curl);
    //关闭URL请求
    curl_close($curl);
    return $info;
}
?>
```

#### 补充

- 其中的 ID 或者 slug 使用命令

```shell
curl -L http://api.travis-ci.org/repos/用户名/仓库  //自己的博客本体的用户名和仓库
```

注意：**就我自己的发现，似乎这个仓库必须是公开的，不然是不能获取 ID 的**。

- 其中的 你的登录 token 在 travis 的网站的 profile 里面获取，直接复制就行了。

#### 新建一个提醒配置

注意 ：新建的是一个通过 API 开始触发的，保持默认的配置即可。获取到了一个 API 链接。

### 语雀配置

- 在知识库里面设置。
- 在开发者选项新建一个 webhook，其中 url 填入上面获得的 API 即可。
