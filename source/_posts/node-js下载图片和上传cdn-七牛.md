---
title: node.js下载图片和上传cdn(七牛)
date: 2018-09-12 15:43:53
tags:
  - node.js
---

项目中有时需要使用第三方到图片，但是如果直接引用第三方到 url，我们将面临一些风险：第一，我们无法保证该地址长期有效；第二，网页中加载该 url 可能无法显示，因为对方很可能做了防盗链处理，这个问题在浏览器端有这个限制但是在后端就没有这个问题了。

所以我们需要通过图片的 url 在服务端将这个图片下载下来保存在服务器上，然后客户端去访问我们自己的服务器，或者把该图片上传到 cdn 上。
<!-- more -->

首先，我们可能需要先下载需要到网络图片，先来个基础到方法：

```bash
const request = require('request');
const fs = require('fs');
const path = require('path');

function downloadUrl(url: string) {
  let filepath = path.join(__dirname, './images/xxx.jpg');
  request(url).pipe(fs.createWriteStream(filepath));
}

// 用以下方式可以监听图片下载成功与否
function downloadUrl(url: string, outputPath: string) {
  let outputStream = fs.createWriteStream(outputPath)
  return new Promise((resolve, reject) => {
    request(url).pipe(outputStream);
    outputStream.on('error', (error) => {
      console.log('downloadImage error', error);
      resolve(false)
    })
    outputStream.on('close', () => {
      console.log('downloadImage 文件下载完成');
      resolve(true)
    });
  })
}
```

如果我们需要把该图片上传到 cdn 上时，我们可以直接传可读 stream（ let readableStream = request(url) ），可以简化下载图片到服务器 - 上传图片到cdn - 删除临时图片这个过程。这里以七牛为例，需要安装 [qiniu](https://github.com/qiniu/nodejs-sdk) 的库。

```bash
const request = require('request');
const fs = require('fs');
const path = require('path');
import * as qiniu from 'qiniu';

// 获取七牛 token
export async function getToken(): Promise<string> {
  const accessKey = CONSTANTS.QINIU.AK;
  const secretKey = CONSTANTS.QINIU.SK;
  const mac = new qiniu.auth.digest.Mac(accessKey, secretKey);
  const putPolicy = new qiniu.rs.PutPolicy({
    scope: CONSTANTS.QINIU.BUCKET
  });
  const uploadToken = putPolicy.uploadToken(mac);
  return uploadToken;
}

export async function upload2CDN(url) {
  let readableStream = request(url);
  const key = `images/xxx_${Date.now()}.jpg`;
  const token = await getToken();
  return new Promise((resolve, reject) => {
    try {
      formUploader.putStream(
        token,
        key,
        readableStream,
        putExtra,
        (respErr, respBody, respInfo) => {
          if (respErr) {
            return resolve(null);
          }
          if (respInfo.statusCode == 200) {
            return resolve(respBody && respBody.key || key);
          } else {
            return resolve(null);
          }
        }
      );
    } catch (e) {
      return resolve(null);
    }
  })
}
```

以上方法只能单次处理一张图片，需要处理多张图片时代码可以参考下面到代码：

```bash
import * as qiniu from 'qiniu';
const config: any = new qiniu.conf.Config();
config.zone = qiniu.zone.Zone_z2;
const formUploader = new qiniu.form_up.FormUploader(config);
const putExtra = new qiniu.form_up.PutExtra();

async uploadFiles(urls: any[], type: string, token?: string) {
  if (!token) {
    token = await get_token();
  }
  let keys = {};
  for (let url of urls) {
    let requestUrl = url.url.trim();
    if (!requestUrl || !/(https?|ftp|file):\/\/\w+\.\w+/.test(requestUrl)) {
      keys[url.id] = null;
      continue
    }
    const uploadFunc = async () => {
      return new Promise((resolve, reject) => {
        let readableStream = request(requestUrl);
        const key = `images/xxx_${Date.now()}.jpg`;
        try {
          formUploader.putStream(
            token,
            key,
            readableStream,
            putExtra,
            (respErr, respBody, respInfo) => {
              if (respInfo.statusCode == 200) {
                return resolve(respBody && respBody.key);
              } else {
                return resolve(null);
              }
            }
          );
        } catch (e) {
          return resolve(null);
        }
      })
    }
    const key = await uploadFunc();
    this.app.logger.info('key: ', key);
    keys[url.id] = key;
  }
  return keys;
}
```