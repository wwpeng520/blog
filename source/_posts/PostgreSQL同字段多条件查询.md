---
title: PostgreSQL同字段多条件查询
date: 2018-07-28 09:40:59
tags:
- Postgresql
- SQL
- 数据库
---
项目有个需求，就是把数据库中的第三方的图片全部传到公司的cdn（七牛）上。但是公司的 cdn 有几个，都需要排除在外，那就需要对某个字段进行多个约束查询了。
先说点题外的，这个需求我的解决过程是（使用官方node版sdk）：

- 查找数据库匹配的图片地址（列表，多数据）
- 调用七牛官方提供的方法获取 token
- 使用 node.js 的 request 方法获取数据流
- 都用七牛数据流上传（表单方式）
- 用七牛上传之后的文件名加上公司的cdn前缀作为最终图片url替换数据库中的数据

<!-- more -->
代码如下：

```bash
import * as qiniu from 'qiniu';
import * as moment from "moment";
const request = require('request');
interface IUploadRetrun {
    id: number;
    url: string | null;
}

async upload_files(urls: IUploadRetrun[], type: string, token?: string) {
    const config: any = new qiniu.conf.Config();
    config.zone = qiniu.zone.Zone_z2;
    const formUploader = new qiniu.form_up.FormUploader(config);
    const putExtra = new qiniu.form_up.PutExtra();
    if (!token) {
        token = await this.get_token(); // 省略 get_token 的代码
    }
    let keys = {};
    for (let url of urls) {
        this.app.logger.info('url: ', url.url)
        const uploadFunc = async () => {
            return new Promise((resolve, reject) => {
                let readableStream = request(url.url);
                const timeStamp = moment().valueOf();
                const key = `images/${type}/${type}_${timeStamp}.jpg`;
                try {
                    formUploader.putStream(
                        token,
                        key,
                        readableStream,
                        putExtra,
                        (respErr, respBody, respInfo) => {
                            if (respInfo.statusCode == 200) {
                                // respBody: { hash: 'FtDNNsaQndsJoye3FRIX_EsLyVnH', key: 'images/avatar/avatar_1532693100439.jpg' }
                                return resolve(respBody && respBody.key);
                            } else {
                                this.app.logger.info('respErr: ', respErr);
                                this.app.logger.info('respBody: ', respBody);
                                return resolve(null);
                            }
                        }
                    );
                } catch (e) {
                    this.app.logger.info(`upload file to qiniu error: ${JSON.stringify(e)}`);
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

因为我的项目使用的是 egg + egg-sequelize(ORM) 的形式，数据库是 PostgreSQL，所以查询的时候我一开始的想法是使用 sequelize 提供的方法，特意查看了一下文档，期望的代码如下：

```bash
let results = await this.app.model.Xxxx.findAll({
    where: {
        image: {
            $notLike: { $any: ['http://cdn.aaa.com%', 'http://cdn.bbb.com%']}
        }
    }
});
```

但是 ts 一直在报错，索性就用 SQL 查询来做了，以下三种方式测试都是可用的。推荐使用第一种方式，语义明确，代码也最简练，第三张是最笨的方法，只需要把所有的约束用 AND 连接就好了。

```bash
SELECT id,image AS url FROM table1 WHERE image NOT SIMILAR TO 'http://(cdn.aaa.com|cdn.bbb.com|cdn.ccc.com)%' ORDER BY id LIMIT 10

SELECT id,image AS url FROM table1 WHERE NOT image LIKE ANY (ARRAY['http://cdn.aaa.com%', 'http://cdn.bbb.com%', 'http://cdn.ccc.com%']) ORDER BY id LIMIT 10

SELECT id,image AS url FROM table1 WHERE (image NOT LIKE 'http://cdn.aaa.com%' AND image NOT LIKE 'http://cdn.bbb.com%' AND image NOT LIKE 'http://cdn.ccc.com%') ORDER BY id LIMIT 10
```

查找并替换图片地址完整代码如下：

```bash
// 查找并替换图片地址
async renew_images(count = 10) {
    let sql = `SELECT id,image AS url FROM table1 WHERE image NOT SIMILAR TO '(${CONTANTS.IMAGE_WHITE_LIST.join("|")})%' ORDER BY id LIMIT ${count}`;
    let images = await this.app.model.query(sql, {
        type: this.app.model.QueryTypes.SELECT
    });
    const token = await this.service.qiniu.get_token();

    let results = await this.service.qiniu.upload_files(images, 'image', token);
    for (let key in results) {
        if (results[key]) {
            this.app.model.HuFen.update({
                image: `http://cdn.xxx.com/${results[key]}`
            }, {
                where: { id: parseInt(key) }
            });
        }
    }
    return results;
}
```
