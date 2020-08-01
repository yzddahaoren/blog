---
title: 利用jsproxy和cloudflare搭建cn用代理
urlname: sic5el
date: 2020-06-12 12:57:34 +0800
tags: []
categories: []
---

![2f437623e0af8372f0b0b1c5e9e2aa30.png](https://cdn.nlark.com/yuque/0/2020/png/1550831/1591938916343-67e6c99b-d80b-4541-9b02-220b7cdad5b6.png#align=left&display=inline&height=175&margin=%5Bobject%20Object%5D&name=2f437623e0af8372f0b0b1c5e9e2aa30.png&originHeight=175&originWidth=606&size=14277&status=done&style=none&width=606)

## 介绍

- 前面的帖子介绍过好用的 cloudflare 的 work 项目
- 可以搭建一个中国本地可用的代理
- 这个代理是 cloudflare 只发数据,js 的数据操作在浏览器端进行,减少服务端压力

## 例子

- [https://hub.yzddahaoren.workers.dev](https://hub.yzddahaoren.workers.dev) //你懂的
- [https://porn.yzddahaoren.workers.dev](https://hub.yzddahaoren.workers.dev) //其他工具网站

## 具体

### 注册账号

- 注册**cloudflare**的账号
- 验证注册的邮箱(必须验证,否则不能用,建议 gmail)

### 创建 work

- 项目地址 : [项目](https://github.com/Berkeley-Reject/workers-proxy/)
- 登陆项目 : [项目](https://workers.cloudflare.com/)
- 点击 ==> **create a worker**
- 修改左上角的一传随机的字母,改成有意义的组合
- 粘贴如下代码 (以下有两种代理)

* 第一种 (工具类)

```jsx
"use strict";

/**
 * static files (404.html, sw.js, conf.js)
 */
const ASSET_URL = "https://etherdream.github.io/jsproxy";

const JS_VER = 10;
const MAX_RETRY = 1;

/** @type {RequestInit} */
const PREFLIGHT_INIT = {
  status: 204,
  headers: new Headers({
    "access-control-allow-origin": "*",
    "access-control-allow-methods":
      "GET,POST,PUT,PATCH,TRACE,DELETE,HEAD,OPTIONS",
    "access-control-max-age": "1728000",
  }),
};

/**
 * @param {any} body
 * @param {number} status
 * @param {Object<string, string>} headers
 */
function makeRes(body, status = 200, headers = {}) {
  headers["--ver"] = JS_VER;
  headers["access-control-allow-origin"] = "*";
  return new Response(body, { status, headers });
}

/**
 * @param {string} urlStr
 */
function newUrl(urlStr) {
  try {
    return new URL(urlStr);
  } catch (err) {
    return null;
  }
}

addEventListener("fetch", (e) => {
  const ret = fetchHandler(e).catch((err) =>
    makeRes("cfworker error:\n" + err.stack, 502)
  );
  e.respondWith(ret);
});

/**
 * @param {FetchEvent} e
 */
async function fetchHandler(e) {
  const req = e.request;
  const urlStr = req.url;
  const urlObj = new URL(urlStr);
  const path = urlObj.href.substr(urlObj.origin.length);

  if (urlObj.protocol === "http:") {
    urlObj.protocol = "https:";
    return makeRes("", 301, {
      "strict-transport-security":
        "max-age=99999999; includeSubDomains; preload",
      location: urlObj.href,
    });
  }

  if (path.startsWith("/http/")) {
    return httpHandler(req, path.substr(6));
  }

  switch (path) {
    case "/http":
      return makeRes("请更新 cfworker 到最新版本!");
    case "/ws":
      return makeRes("not support", 400);
    case "/works":
      return makeRes("it works");
    default:
      // static files
      return fetch(ASSET_URL + path);
  }
}

/**
 * @param {Request} req
 * @param {string} pathname
 */
function httpHandler(req, pathname) {
  const reqHdrRaw = req.headers;
  if (reqHdrRaw.has("x-jsproxy")) {
    return Response.error();
  }

  // preflight
  if (
    req.method === "OPTIONS" &&
    reqHdrRaw.has("access-control-request-headers")
  ) {
    return new Response(null, PREFLIGHT_INIT);
  }

  let acehOld = false;
  let rawSvr = "";
  let rawLen = "";
  let rawEtag = "";

  const reqHdrNew = new Headers(reqHdrRaw);
  reqHdrNew.set("x-jsproxy", "1");

  // 此处逻辑和 http-dec-req-hdr.lua 大致相同
  // https://github.com/EtherDream/jsproxy/blob/master/lua/http-dec-req-hdr.lua
  const refer = reqHdrNew.get("referer");
  const query = refer.substr(refer.indexOf("?") + 1);
  if (!query) {
    return makeRes("missing params", 403);
  }
  const param = new URLSearchParams(query);

  for (const [k, v] of Object.entries(param)) {
    if (k.substr(0, 2) === "--") {
      // 系统信息
      switch (k.substr(2)) {
        case "aceh":
          acehOld = true;
          break;
        case "raw-info":
          [rawSvr, rawLen, rawEtag] = v.split("|");
          break;
      }
    } else {
      // 还原 HTTP 请求头
      if (v) {
        reqHdrNew.set(k, v);
      } else {
        reqHdrNew.delete(k);
      }
    }
  }
  if (!param.has("referer")) {
    reqHdrNew.delete("referer");
  }

  // cfworker 会把路径中的 `//` 合并成 `/`
  const urlStr = pathname.replace(/^(https?):\/+/, "$1://");
  const urlObj = newUrl(urlStr);
  if (!urlObj) {
    return makeRes("invalid proxy url: " + urlStr, 403);
  }

  /** @type {RequestInit} */
  const reqInit = {
    method: req.method,
    headers: reqHdrNew,
    redirect: "manual",
  };
  if (req.method === "POST") {
    reqInit.body = req.body;
  }
  return proxy(urlObj, reqInit, acehOld, rawLen, 0);
}

/**
 *
 * @param {URL} urlObj
 * @param {RequestInit} reqInit
 * @param {number} retryTimes
 */
async function proxy(urlObj, reqInit, acehOld, rawLen, retryTimes) {
  const res = await fetch(urlObj.href, reqInit);
  const resHdrOld = res.headers;
  const resHdrNew = new Headers(resHdrOld);

  let expose = "*";

  for (const [k, v] of resHdrOld.entries()) {
    if (
      k === "access-control-allow-origin" ||
      k === "access-control-expose-headers" ||
      k === "location" ||
      k === "set-cookie"
    ) {
      const x = "--" + k;
      resHdrNew.set(x, v);
      if (acehOld) {
        expose = expose + "," + x;
      }
      resHdrNew.delete(k);
    } else if (
      acehOld &&
      k !== "cache-control" &&
      k !== "content-language" &&
      k !== "content-type" &&
      k !== "expires" &&
      k !== "last-modified" &&
      k !== "pragma"
    ) {
      expose = expose + "," + k;
    }
  }

  if (acehOld) {
    expose = expose + ",--s";
    resHdrNew.set("--t", "1");
  }

  // verify
  if (rawLen) {
    const newLen = resHdrOld.get("content-length") || "";
    const badLen = rawLen !== newLen;

    if (badLen) {
      if (retryTimes < MAX_RETRY) {
        urlObj = await parseYtVideoRedir(urlObj, newLen, res);
        if (urlObj) {
          return proxy(urlObj, reqInit, acehOld, rawLen, retryTimes + 1);
        }
      }
      return makeRes(res.body, 400, {
        "--error": `bad len: ${newLen}, except: ${rawLen}`,
        "access-control-expose-headers": "--error",
      });
    }

    if (retryTimes > 1) {
      resHdrNew.set("--retry", retryTimes);
    }
  }

  let status = res.status;

  resHdrNew.set("access-control-expose-headers", expose);
  resHdrNew.set("access-control-allow-origin", "*");
  resHdrNew.set("--s", status);
  resHdrNew.set("--ver", JS_VER);

  resHdrNew.delete("content-security-policy");
  resHdrNew.delete("content-security-policy-report-only");
  resHdrNew.delete("clear-site-data");

  if (
    status === 301 ||
    status === 302 ||
    status === 303 ||
    status === 307 ||
    status === 308
  ) {
    status = status + 10;
  }

  return new Response(res.body, {
    status,
    headers: resHdrNew,
  });
}

/**
 * @param {URL} urlObj
 */
function isYtUrl(urlObj) {
  return (
    urlObj.host.endsWith(".googlevideo.com") &&
    urlObj.pathname.startsWith("/videoplayback")
  );
}

/**
 * @param {URL} urlObj
 * @param {number} newLen
 * @param {Response} res
 */
async function parseYtVideoRedir(urlObj, newLen, res) {
  if (newLen > 2000) {
    return null;
  }
  if (!isYtUrl(urlObj)) {
    return null;
  }
  try {
    const data = await res.text();
    urlObj = new URL(data);
  } catch (err) {
    return null;
  }
  if (!isYtUrl(urlObj)) {
    return null;
  }
  return urlObj;
}
```

- 第二种 (你懂的)

```jsx
// Website you intended to retrieve for users.
const upstream = "www.pornhub.com";

// Custom pathname for the upstream website.
const upstream_path = "/";

// Website you intended to retrieve for users using mobile devices.
const upstream_mobile = "www.pornhub.com";

// Countries and regions where you wish to suspend your service.
const blocked_region = ["KP", "SY", "PK", "CU"];

// IP addresses which you wish to block from using your service.
const blocked_ip_address = ["0.0.0.0", "127.0.0.1"];

// Whether to use HTTPS protocol for upstream address.
const https = true;

// Whether to disable cache.
const disable_cache = false;

// Replace texts.
const replace_dict = {
  $upstream: "$custom_domain",
  "//cn.phncdn.com": "//hub.yzddahaoren.workers.dev", // replace this with your other deployment.
  "var scriptText": "", // Issue #30
  "trafficjunky.com": "",
  "trafficjunky.net": "",
  "contentabc.com": "",
  "Ads By Traffic Junky": "",
  "cdn1d-static-shared.phncdn.com/iframe-1.1.5.html": "",
  "/ads/iframe-mobile-3.0.0.html": "",
  "<iframe": "<!--",
  "</iframe>": "-->",
  "Remove Ads": "",
};

addEventListener("fetch", (event) => {
  event.respondWith(fetchAndApply(event.request));
});

async function fetchAndApply(request) {
  const region = request.headers.get("cf-ipcountry").toUpperCase();
  const ip_address = request.headers.get("cf-connecting-ip");
  const user_agent = request.headers.get("user-agent");

  let response = null;
  let url = new URL(request.url);
  let url_hostname = url.hostname;

  if (https == true) {
    url.protocol = "https:";
  } else {
    url.protocol = "http:";
  }

  if (await device_status(user_agent)) {
    var upstream_domain = upstream;
  } else {
    var upstream_domain = upstream_mobile;
  }

  url.host = upstream_domain;
  if (url.pathname == "/") {
    url.pathname = upstream_path;
  } else {
    url.pathname = upstream_path + url.pathname;
  }

  if (blocked_region.includes(region)) {
    response = new Response(
      "Access denied: WorkersProxy is not available in your region yet.",
      {
        status: 403,
      }
    );
  } else if (blocked_ip_address.includes(ip_address)) {
    response = new Response(
      "Access denied: Your IP address is blocked by WorkersProxy.",
      {
        status: 403,
      }
    );
  } else {
    let method = request.method;
    let request_headers = request.headers;
    let new_request_headers = new Headers(request_headers);

    new_request_headers.set("Host", upstream_domain);
    new_request_headers.set("Referer", url.protocol + "//" + url_hostname);

    let original_response = await fetch(url.href, {
      method: method,
      headers: new_request_headers,
    });

    connection_upgrade = new_request_headers.get("Upgrade");
    if (connection_upgrade && connection_upgrade.toLowerCase() == "websocket") {
      return original_response;
    }

    let original_response_clone = original_response.clone();
    let original_text = null;
    let response_headers = original_response.headers;
    let new_response_headers = new Headers(response_headers);
    let status = original_response.status;

    if (disable_cache) {
      new_response_headers.set("Cache-Control", "no-store");
    }

    new_response_headers.set("access-control-allow-origin", "*");
    new_response_headers.set("access-control-allow-credentials", true);
    new_response_headers.delete("content-security-policy");
    new_response_headers.delete("content-security-policy-report-only");
    new_response_headers.delete("clear-site-data");

    if (new_response_headers.get("x-pjax-url")) {
      new_response_headers.set(
        "x-pjax-url",
        response_headers
          .get("x-pjax-url")
          .replace("//" + upstream_domain, "//" + url_hostname)
      );
    }

    const content_type = new_response_headers.get("content-type");
    if (
      content_type != null &&
      content_type.includes("text/html") &&
      content_type.includes("UTF-8")
    ) {
      original_text = await replace_response_text(
        original_response_clone,
        upstream_domain,
        url_hostname
      );
    } else {
      original_text = original_response_clone.body;
    }

    response = new Response(original_text, {
      status,
      headers: new_response_headers,
    });
  }
  return response;
}

async function replace_response_text(response, upstream_domain, host_name) {
  let text = await response.text();

  var i, j;
  for (i in replace_dict) {
    j = replace_dict[i];
    if (i == "$upstream") {
      i = upstream_domain;
    } else if (i == "$custom_domain") {
      i = host_name;
    }

    if (j == "$upstream") {
      j = upstream_domain;
    } else if (j == "$custom_domain") {
      j = host_name;
    }

    let re = new RegExp(i, "g");
    text = text.replace(re, j);
  }
  return text;
}

async function device_status(user_agent_info) {
  var agents = [
    "Android",
    "iPhone",
    "SymbianOS",
    "Windows Phone",
    "iPad",
    "iPod",
  ];
  var flag = true;
  for (var v = 0; v < agents.length; v++) {
    if (user_agent_info.indexOf(agents[v]) > 0) {
      flag = false;
      break;
    }
  }
  return flag;
}
```

- 简洁 :
  - 第一个是包含 **google youtube twitter facebook github **和其他,其中**twitter**已经不能用了
  - 第二种可以换成自己需要的网站,具体的项目内自己看,前面有项目地址,这里是**pornhub**

\*\*

## 其他

1. 这里的速度比较不快,但是还能用
1. 其中第一个项目的 : [链接 ]()
1. 这个项目可以自己用自己的国外**vps**搭建
