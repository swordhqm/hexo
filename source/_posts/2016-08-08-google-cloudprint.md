---
title: google_cloudprint
date: 2016-08-08 15:01:58
tags:
    - open
category::
    - 技术
---

# cloudprint

初衷，解决直接打印问题。寻求路径，jatoolsprinter, lodop. 怎么说呢，用chrome 去试着安装jatoolsprinter 插件，但是看起来并不work。lodop 有试过，看起来是ok的，非免费，大致机理，创建了一个local server，然后web 向local server交互，由local server直接和打印机交互，免费情况下，打印的东东底下会有水印。若干时间后，注意到cloudprint，google cloudprint，可以直接通过[chrome device](chrome://devices/), 添加设备到Google 云打印。

    [首页](https://www.google.com/cloudprint/learn/printers.html)
    看起来有些打印机可以直接连接网络然后通过配置连接到Google 云打印
    传统打印机可以通过PC安装打印机驱动，然后通过PC 添加到Google 云打印
    不知道机理，如果浏览器无法直接控制打印机，那么Google 云打印可能是通过打印机驱动和远端server交互，然后app或者browser通过server和打印机交互。但是，对于Google cloud ready的设备也就是直接连接到云的设备可能是设备可以直接和server交互。

# api

目前云端用户界面看起来还是测试版，而且竟然在API console看不到 cloudprint相关的东西

[云端用户界面](https://www.google.com/cloudprint#printers)
[cloudprint API](https://developers.google.com/cloud-print/)
[API console](https://console.developers.google.com/apis/credentials?project=cloudprinter-1470324129369)

目前找到一种方法能够走通cloudprint api。

    方式：
    1. 手动在cloudprint 上面添加打印机，虽然点高级信息能看到printerid 和 proxy，但是由于没有auth2.0 认证，直接通过curl去访问是会fail，比如：
    curl "https://www.google.com/cloudprint/getauthcode?printerid=26bce0fb-29d0-8bed-50c5-e23e76acb2ab&oauth_client_id=48642476290-sbilb9g5o8lce06b2flecgblvh6q6a2n.apps.googleusercontent.com" -H "X-CloudPrint-Proxy" -d ""
    2. 在API console 创建auth2.0 凭证，这里会有client id，secret key， 解决的是第三方服务和 service的信任问题，当用户在第三方 请求 service的资源，那么第三方会用这个凭证去向service 请求认证授权，用户授权后就会拿到authorization code.
    3. 关于auth2.0认证，这里简单描述 auth2.0是解决用户，service，和第三方的信任问题。
    用到 https://github.com/google/google-api-nodejs-client
    步骤，先通过client id, secret key获取到认证url，然后通过url拿到authorization code.
        CLIENT_ID = "48642476290-sbilb9g5o8lce06b2flecgblvh6q6a2n.apps.googleusercontent.com";
        CLIENT_SECRET = "CZijBuzzUQk6v9rS8f0meCLf";
        REDIRECT_URL = "urn:ietf:wg:oauth:2.0:oob";
        var google = require('googleapis');
        var OAuth2 = google.auth.OAuth2;
        var oauth2Client = new OAuth2(CLIENT_ID, CLIENT_SECRET, REDIRECT_URL);
        var scopes = [ 
            'https://www.googleapis.com/auth/cloudprint'
        ];
        var url = oauth2Client.generateAuthUrl({
            access_type:'offline',
            scope:scopes
        })
        至此获取到请求认证url，然后通过open 这个链接，用户授权后会拿到authorization code: '4/ZjqYuMi04uH2KTK7Vd9ylEIbUnaeVLypJiZN7Bi2lKg'.通过authorization code去拿token
        oauth2Client.getToken('4/ZjqYuMi04uH2KTK7Vd9ylEIbUnaeVLypJiZN7Bi2lKg',  function(err, tokens){
            if(!err) {
                oauth2Client.setCredentials(tokens);
                console.log(tokens)
            }   
        });
        access_token: 'ya29.Ci85A7At7FoMg2DklFU6huDvywYAS6-XZEfxSkVPACGv8K3McxZn2WFgwuG-drXMMQ',
        token_type: 'Bearer',
        expires_in: 3600,
        refresh_token: '1/kC2xybjLrAs3Y8Hv1VCjQS9aPBTv1EbQ9fyrzghHxEw'
        拿到的token里会有access_token和refresh_token。 access_token有有效期，不过可以通过refresh_token去刷新。然后就可以通过nodejs 的第三方cloudprint 去测试api