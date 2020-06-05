---
title: kkkk
tags:
  - null
categories:
  - null
date: 2020-06-05 22:38:56
cover:
mathjax:
katex:
hide:
---
使用yuque-hexo
参考文档 链接
事先拥有一个 hexo 项目，并在 package.json 中配置相关信息，可参考 如下。
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.9.0"
  },
  "yuqueConfig": {
    "postPath": "source/_posts/yuque",
    "cachePath": "yuque.json",
    "mdNameFormat": "title",
    "adapter": "hexo",
    "concurrency": 5,
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "gpfer",
    "repo": "blog",
    "onlyPublished": false
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
  "dependencies": {
    "core-js": "^3.1.4",
    "eslint": "^4.19.1",
    "hexo": "^3.9.0",
    "hexo-deployer-git": "^1.0.0",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index-pin-top": "^0.2.2",
    "hexo-generator-search": "^2.4.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^1.0.1",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.3"
  },
  "devDependencies": {
    "yuque-hexo": "^1.6.1"
  }
}
复制代码serverless
使用的是腾讯云的云函数服务
创建一个云函数,地区选择北京或者广州，之前创建函数的时候 其他地区不能使用触发器
基本配置

index.php
<?php
function main_handler($event, $context) {
    // 解析语雀post的数据
    $update_title = '';
    if($event->body){
        $yuque_data= json_decode($event->body);
        $update_title .= $yuque_data->data->title;
    }

    // default params
    $repos = '*****';  // 你的仓库id 或 slug
    $token = '******'; // 你的登录token
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
复制代码触发方式


作者：北枳
链接：https://juejin.im/post/5ddf89f5f265da05e04bf64a
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
