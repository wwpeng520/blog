---
title: node.js下生成带二维码的分享图片（图片合成）
date: 2018-09-13 21:15:54
tags:
  - node.js
---

APP 开发，后台（Node.js）需要生成带有二维码的分享图片，底图是一张活动页面，在底图的中上方放置与分享者有关的二维码。即把两张图片合成为一张图片。

任务流程如下：

1. 生成二维码
2. 合成图片
3. 上传图片至 cdn

需要用到工具：[node-qrcode](https://github.com/soldair/node-qrcode#qr-code-options), [gm](https://github.com/aheckmann/gm), [qiniu](https://github.com/qiniu/nodejs-sdk)

<!-- more -->

## 生成二维码

二维码指向的是带有用户信息的 url，例如：`https://aaa.bbb.com/share?invite_code=XYZABZ`，这个“invite_code”就是每个用户的邀请码。扫描该二维码本质就是引导跳转到该 url。
因为七牛可以使用数据流上传方式上传，如果需要生成二维码，我们可以直接生成 stream，然后上传，而简化了「保存二维码到本地 - 上传本地图片至 cdn - 删除本地图片」的流程。

```typescript
import * as QRCode from "qrcode";
import * as qiniu from "qiniu";
import { Duplex } from "stream";
const request = require("request");

// 保存二维码到本地
export async function createQRCodeToFile(
  path: string,
  url: string,
  option?: any
): Promise<boolean> {
  return new Promise<boolean>((resolve, reject) => {
    QRCode.toFile(path, url, option || {}, function(err) {
      if (!err) {
        console.log("done!");
        return resolve(true);
      } else {
        console.log("error!");
        return resolve(false);
      }
    });
  });
}

// 生成 stream，可直接上传 cdn
export async function createQRCodeToStream(url: string) {
  const base64 = await QRCode.toDataURL(url); // base64
  let buffer = new Buffer(base64.replace(/.+,/, ""), "base64");
  let stream = new Duplex();
  stream.push(buffer);
  stream.push(null);
  return stream;
}
```

## 合成图片

合成图片的库有几个：[node-imagemagick-native](https://github.com/elad/node-imagemagick-native), [gm](https://github.com/aheckmann/gm)，这里选用 star 数更高的 gm 库。
安装 gm 库需要安装 imagemagick 和 graphicsmagick，安装步骤请参考[这里](https://github.com/aheckmann/gm#getting-started)。gm 库本质上是通过执行这两个软件的相关命令来对图片进行操作。

gm 文档的 api 特别的多，如调整图片大小、颜色、形状，给图片加各种效果，加文字，比较两张图片异同等。这里我只需要使用合成图片的方法。大概看看发现合成图片的方法也有几种：composite、mosaic、append、draw。我使用了 draw 方法，其他方法我 copy 一下别人的代码做个演示。

1. mosaic 方法

> create a mosaic from an image or an image sequence

从字面量来看是：从图像或图像序列创建马赛克。这个没深究，可以参考[艾弗尤](https://blog.csdn.net/af52520/article/details/77971653)同学的文章。（以下代码也是 copy 自这里）

```js
var gm = require("gm");

gm()
  .in("-page", "+0+0") //-page是设置图片位置，所有的图片以左上为原点，向右、向下为正
  .in("Images/bg.png") //底图，到这里第一张图就设置完了，要先设置参数，再设置图片
  .in("-resize", "200x200") //设置微信二维码图片的大小（等比缩放）
  .in("-page", "+100+100") //设置微信二维码图片的位置
  .in("Images/qrcode.png") //二维码图
  .in("-page", "+75+75") //logo图位置
  .in("Images/logo.png") //logo图
  .mosaic() //图片合成
  .write("Images/final.png", function(err) {
    //图片写入
    if (!!err) {
      console.log(err);
    } else {
      console.log("ok");
    }
  });
```

1. draw 方法

> annotate an image with one or more graphic primitives.
> Use this option to annotate an image with one or more graphic primitives. The primitives include shapes, text, transformations, and pixel operations. The shape primitives are

point x,y
line x0,y0 x1,y1
rectangle x0,y0 x1,y1
roundRectangle x0,y0 x1,y1 wc,hc
arc x0,y0 x1,y1 a0,a1
ellipse x0,y0 rx,ry a0,a1
circle x0,y0 x1,y1
polyline x0,y0 ... xn,yn
polygon x0,y0 ... xn,yn
Bezier x0,y0 ... xn,yn
path path specification
image operator x0,y0 w,h filename <<---

从所提供的参数来看我们需要按照最后一条规则来使用：composite operator, image location, image size, filename
例如：'image Over 200, 200, 100, 100 "./abc.jpg"'。代表以 'image Over' 方式合成，被合成图片的坐标是‘200, 200’，被合成图片大小‘100, 100’，图片路径是"./abc.jpg"。参考代码如下

```ts
let params = "image Over 250, 170, 250, 250";

export async function composeLocalImages(
  bgImage: string,
  qrcode: string,
  params: string,
  outputPath: string
): Promise<boolean> {
  return new Promise<boolean>((resolve, reject) => {
    gm(bgImage)
      .draw(`${params} "${qrcode}"`)
      .write(outputPath, function(err) {
        if (!err) {
          console.log("composeLocalImages done!");
          return resolve(true);
        } else {
          console.log(err.message || "composeLocalImages 出错了！");
          return resolve(false);
        }
      });
  });
}
```

## 上传图片至 cdn

上传图片至七牛，我在另外一篇文章中也写过，这里不做赘述了，直接贴代码：（以下使用的是 egg 的框架）

```ts
import * as imageUtils from '../utils/image';
const { exec } = require('child_process');
const tempDir = './temp_images';

async createInviteCard(inviteCode: string) {
    let bgImage = `${tempDir}/bg.png`; // 背景图
    await imageUtils.dirExists(tempDir); // tempDir 目录不存在时新建该目录

    let now = Date.now();
    let cardCDNKey = `images/card_${now}.png`;
    let qrcodeCDNKey = `images/qrcode_${now}.png`;
    let cardOutputPath = `${tempDir}/invite_card_${now}.jpg`;
    // 合成参数：composite operator, image location, image size(, and filename 输出路径)
    let params = 'image Over 250, 170, 250, 250';

    const qrcodeOutputPath = await this.createAndSaveInviteQRCode(inviteCode);
    if (!qrcodeOutputPath) {
      return this.ctx.throw(400, "保存二维码失败！");
    }

    const composeResult = await imageUtils.composeLocalImages(
      bgImage,
      qrcodeOutputPath,
      params,
      cardOutputPath
    );
    if (!composeResult) {
      return this.ctx.throw(400, "合成邀请图片失败！");
    }

    // 上传至 cdn
    const qrcodeCDNUrl = await imageUtils.uploadFileToCDN(qrcodeCDNKey, qrcodeOutputPath);
    const cardCDNUrl = await imageUtils.uploadFileToCDN(cardCDNKey, cardOutputPath);
    if (!qrcodeCDNUrl) {
      return this.ctx.throw(400, "邀请二维码上传CDN失败！");
    }
    if (!cardCDNUrl) {
      return this.ctx.throw(400, "邀请卡片上传CDN失败！");
    }
    // 删除临时图片
    exec('rm -f ' + qrcodeOutputPath);
    exec('rm -f ' + cardOutputPath);

    return { qrcodeCDNUrl, cardCDNUrl };
  }
```

生成图片如下：
![分享二维码]](/images/common-articles/invite_card_1.jpg)

参考文章

- [gm](https://github.com/aheckmann/gm)
- [node 服务器如何生成有 logo 和背景的带参数二维码](https://blog.csdn.net/af52520/article/details/77971653)
- [Nodejs 图片编辑和中文乱码](https://www.jianshu.com/p/a651258c9135?_wv=5)
- [node图片合成](https://laclys.github.io/2018/03/10/node%E5%9B%BE%E7%89%87%E5%90%88%E6%88%90/)
