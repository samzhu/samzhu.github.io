---
title: Receive SendGrid inbound email parse webhook
date: 2021-02-23 09:20:26
tags:
  - SaaS
  - Nodejs
  - Email
categories:
  - Cloud
---

完成了發信功能後, 偶爾還是會有客戶會想透過 Email 聯繫, 所以還是要有個客服信箱來收信, SendGrid 也可以代為收信, 但他不像傳統 mailserver 信件會儲存起來等你上線去收, 而是提供 parse webhook 來讓你自己接收信件內容.

<!--more-->

## Setting up an MX Record
首先你要在 DNS 上設定 MX Record 如下
![https://i.imgur.com/Y2Onu4p.png](https://i.imgur.com/Y2Onu4p.png)

設定完如下 (If there is no field for priority, type 10 before the address. e.g. 10 mx.sendgrid.net.)
![https://i.imgur.com/FbvPd6q.jpg](https://i.imgur.com/FbvPd6q.jpg)

## 使用 Cloud function 接收 webhook
使用 cloud Function 的檔案如下  

package.json
``` json
{
  "name": "sample-http",
  "version": "0.0.1",
  "dependencies": {
    "@google-cloud/storage": "^5.0.0",
    "busboy": "^0.3.0",
    "escape-html": "^1.0.3"
  }
}
```

index.js
``` js

const Busboy = require('busboy');

/**
 * Responds to any HTTP request.
 *
 * @param {!express:Request} req HTTP request context.
 * @param {!express:Response} res HTTP response context.
 */
exports.receiveParsingEmail = (req, res) => {
    var http = require("https");
    console.log("Cloud Function Start : " + req.body.toString());

    let apikey = process.env.apikey;
    let fromaddr = process.env.fromaddr;
    let fromname = process.env.fromname;
    let toaddr = process.env.toaddr;
    let toname = process.env.toname;
    // let mailcontent = req.body.toString();

    console.log("Mail to addr=" + req.body.toString());

    var options = {
        "method": "POST",
        "hostname": "api.sendgrid.com",
        "port": null,
        "path": "/v3/mail/send",
        "headers": {
            "authorization": "Bearer " + apikey,
            "content-type": "application/json"
        }
    };

    var mailreq = http.request(options, function (mailres) {
        var chunks = [];
        mailres.on("data", function (chunk) {
            chunks.push(chunk);
        });
        mailres.on("end", function () {
            var body = Buffer.concat(chunks);
            console.log(body.toString());
        });
    });

    const busboy = new Busboy({ headers: req.headers });
    const fields = {};
    // This code will process each non-file field in the form.
    busboy.on('field', (fieldname, val) => {
        console.log(`Processed field ${fieldname}: ${val}.`);
        fields[fieldname] = val;
    });

    busboy.on('finish', async () => {
        let mailcontent = 'subject:' + '\r\n' + fields.subject + '\r\n';
        mailcontent = mailcontent + '\r\n' + 'from:' + fields.from + '\r\n';
        mailcontent = mailcontent + '\r\n' + 'content:' + fields.text;

        mailreq.write(JSON.stringify({
            personalizations: [{
                to: [{
                    email: toaddr,
                    name: toname
                }],
                subject: "mail notify"
            }],
            content: [{
                type: "text/plain",
                value: mailcontent
            }],
            from: {
                email: fromaddr,
                name: fromname
            }
        }));
        mailreq.end();

        res.status(200).send("OK");
    });
    busboy.end(req.rawBody);
};
```

以上範例 就可以完成 接收 webhook 再將內容轉發到我們指定的信箱

## References
[Getting started with the SendGrid API](https://sendgrid.com/docs/for-developers/sending-email/api-getting-started/)  
[Setting Up The Inbound Parse Webhook](https://sendgrid.com/docs/for-developers/parsing-email/setting-up-the-inbound-parse-webhook/)  
[Cloud Functions HTTP parse multipart/form-data](https://cloud.google.com/functions/docs/samples/functions-http-form-data#functions_http_form_data-nodejs)  
  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記 </span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samzhu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">創用CC 姓名標示-非商業性-相同方式分享 4.0 國際 授權條款</a>釋出。