---
GA: UA-139154617-3
disqus: ericstudio
langs: zh-tw
---
# 匯入公開發行的憑証到Azure Key Vault

#### 一般情況下公開發行的憑證會有三層
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210221135820.png)

#### 所以要準備以下幾種憑證的資料
+ 1.憑證本人的private key(一般檔名是憑證.key)
+ 2.憑證本人的public key(一般檔名是憑證.crt)
+ 3.相關憑證鏈(憑證的爸爸跟阿公)的public key

以上必須是沒有**密碼保護**PEM格式，把上面的內容以下列的格式合併成一個檔案
```
-----BEGIN PRIVATE KEY-----
(憑證本人的private key)
-----END PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
(憑證本人的public key)
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
(憑證的爸爸，中繼憑證)
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
(憑證的阿公，RootCA)
-----END CERTIFICATE-----
```
就可以上傳到Azure Key Vault了。

如果你原始的憑證是pfx格式，可以參考保哥這篇 [如何在收到 PFX 或 CER 憑證檔之後使用 OpenSSL 進行常見的格式轉換](https://blog.miniasp.com/post/2019/04/17/Convert-PFX-and-CER-format-using-OpenSSL) 來取拿到憑證.key(private key)跟憑證.crt(public key)的檔案


###### tags: `Azure` `Azure Key Vault`