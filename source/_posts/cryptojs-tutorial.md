---
title: 前端加密JS库--CryptoJS 使用指南
date: 2018-08-11 12:07:39
tags:
    - cryptojs
categories:
    - javascript
---

有时候项目涉及到的敏感数据比较多，为了信息安全，我们常常需要对一些数据进行接口加密处理，如编码、将明文转化为暗文、加密比对、AES + BASE64 算法加密等。
接下来我们就分别说一下 CryptoJS 常用的一些方法。

[CryptoJS文档](https://github.com/brix/crypto-js)

### Base64 编码

**为什么要编码？**

由于一些网络通讯协议的限制, 又或者是出于信息加密的目的, 我们就需要将原信息转换为base64编码,然后才能进行传输.例如，发送某些含有 ASCII 码表中0到31之间的控制字符的数据。

**window.btoa 对字符串进行 base64编码（注意不能编码中文）;**
**winodw.atob 对 base64字符串 进行解码（对于包含中文的 base64编码，不能正确解码）;**

通常的方法是通过 window.btoa() 方法对源数据进行编码, 然后接收方使用 window.atob() 方法对其进行解码, 从而得到原数据。但是这种方法存在的问题是：window.btoa() 不支持中文, window.atob() 转换含有中文的 base64编码 的时候中文部分会变为乱码。另一个存在的问题是解码github的readme数据的时候也是乱码。经过查找相关资料发现了 Base64的编码与解码转的最优方案:

``` js
// 编码
function utf8_to_b64(str) {
    return window.btoa(unescape(encodeURIComponent(str)));
}

// 解码
function b64_to_utf8(str) {
    return decodeURIComponent(escape(window.atob(str)));
}

// Usage:
utf8_to_b64('✓ à la mode'); // 4pyTIMOgIGxhIG1vZGU=
b64_to_utf8('4pyTIMOgIGxhIG1vZGU='); // "✓ à la mode"

utf8_to_b64('I \u2661 Unicode!'); // SSDimaEgVW5pY29kZSE=
b64_to_utf8('SSDimaEgVW5pY29kZSE='); // "I ♡ Unicode!"

utf8_to_b64('我爱中国'); // 5oiR54ix5Lit5Zu9
b64_to_utf8('SSDimaEgVW5pY29kZSE='); // "我爱中国"

utf8_to_b64(123456); // MTIzNDU2
b64_to_utf8("MTIzNDU2"); // 123456
```

### AES 加密

**安装**

``` bash
$ npm install crypto-js
```

**aes加密: crypto.js**
```js
import CryptoJS from "crypto-js";

const key = CryptoJS.enc.Utf8.parse("1234567890000000"); //16位
const iv = CryptoJS.enc.Utf8.parse("1234567890000000");

export default {
  //aes加密
  encrypt(word) {
    let encrypted = "";
    if (typeof word == "string") {
      const srcs = CryptoJS.enc.Utf8.parse(word);
      encrypted = CryptoJS.AES.encrypt(srcs, key, {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
      });
    } else if (typeof word == "object") {
      //对象格式的转成json字符串
      const data = JSON.stringify(word);
      const srcs = CryptoJS.enc.Utf8.parse(data);
      encrypted = CryptoJS.AES.encrypt(srcs, key, {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
      });
    }
    return encrypted.ciphertext.toString();
  },
  // aes解密
  decrypt(word) {
    const encryptedHexStr = CryptoJS.enc.Hex.parse(word);
    const srcs = CryptoJS.enc.Base64.stringify(encryptedHexStr);
    const decrypt = CryptoJS.AES.decrypt(srcs, key, {
      iv: iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    });
    const decryptedStr = decrypt.toString(CryptoJS.enc.Utf8);
    return decryptedStr.toString();
  }
};
```
**使用**
```js
import Crypto from "@/utils/crypto";

Crypto.encrypt("✓ à la mode"); // b915bf40c4795227488da86978f55fce
Crypto.decrypt(userPwd); // "✓ à la mode"

Crypto.encrypt("✓ à la mode"); // 6317313288b32bf1909f165ec530d60a
Crypto.decrypt(userPwd); // "I ♡ Unicode!"

Crypto.encrypt("我爱中国"); // 1898a34273855f55255437aa22f87504
Crypto.decrypt(userPwd); // "我爱中国"

Crypto.encrypt("123456"); // dd7a6c4edc68e683b700a7a2846a2bc6
Crypto.decrypt(userPwd); // "123456"
```

### 前后端数据通信参数加密

加密代码实现:

项目中需要将所有传到后台的参数分5个步骤处理：

第一步：排序

第二步：将排序好的参数进行MD5加密作为接口的签名

第三步：将排序好的参数和接口签名拼接上进行AES加密

第四部：将AES加密后的密文Base64加密

第五步：将最终的密文encodeURIComponent；

代码如下：

```js
function encryption(data) {
    let strs=[];
    for(let i in data){
        strs.push(i+'='+data[i]);
    }
    strs.sort();  // 数组排序
    strs=strs.join('&'); // 数组变字符串
    let endData=strs+'&sign='+CryptoJS.MD5(strs+'ADfj3kcadc2349akvm1CPFFCD84f').toString(); // MD5加密
    let key = CryptoJS.enc.Utf8.parse("0880076B18D7EE81"); // 加密秘钥
    let iv = CryptoJS.enc.Utf8.parse("CB3EC842D7C69578");  //  矢量
    let encryptResult = CryptoJS.AES.encrypt(endData,key, {   //  AES加密
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7  // 后台用的是pad.Pkcs5,前台对应为Pkcs7
    });
    return encodeURIComponent(CryptoJS.enc.Base64.stringify(encryptResult.ciphertext));  // Base64加密encode;
}
```

加密最终的密文拼接在接口地址后面，请求接口。后台返回的数据也是密文；解密方法如下：

```js
function decryption(data) {
    let key = CryptoJS.enc.Utf8.parse("0880076B18D7EE81");  // 加密秘钥
    let iv = CryptoJS.enc.Utf8.parse("CB3EC842D7C69578");   //  矢量
    let baseResult=CryptoJS.enc.Base64.parse(data);   // Base64解密
    let ciphertext=CryptoJS.enc.Base64.stringify(baseResult);     // Base64解密
    let decryptResult = CryptoJS.AES.decrypt(ciphertext,key, {    //  AES解密
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    });
    let resData=decryptResult.toString(CryptoJS.enc.Utf8).toString();

    return JSON.parse(resData);
}
```
备注：因为后台返回的数据是json格式；所以做种return的时候使用JSON.parse();如果解密的目标为字符串，比如说需要解密的是一个加密的token值。那就要做相应的改动：
```js
function decryption(data) {
    let key = CryptoJS.enc.Utf8.parse("0880076B18D7EE81");  // 加密秘钥
    let iv = CryptoJS.enc.Utf8.parse("CB3EC842D7C69578");   //  矢量
    let baseResult=CryptoJS.enc.Base64.parse(data);   // Base64解密
    let ciphertext=CryptoJS.enc.Base64.stringify(baseResult);     // Base64解密
    let decryptResult = CryptoJS.AES.decrypt(ciphertext,key, {    //  AES解密
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    });
    return CryptoJS.enc.Utf8.stringify(decryptResult);
}
```
