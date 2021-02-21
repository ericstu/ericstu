---
GA: UA-139154617-3
disqus: ericstudio
langs: zh-tw
---
# 在Azure Container Registry （ACR） 啟用自訂網域

現階段(2021年2月)，微軟在Github有提供一份文件 [Using Custom Domains with Azure Container Registry](https://github.com/Azure/acr/blob/main/docs/custom-domain/README.md) 。

我們自己做完一些基本設定之後**還要再找Azure Support開通**。但其實這份文件也沒有很精確，而且有一點小錯誤。

底下我把我做的步驟整理出來。

## 1.Azure環境

+ ACR的SKU必須是 **進階** 的等級。
+ 需要有 Azure KeyVault


## 2.ACR 相關設定

### 開啟系統指派的受控識別
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218000201.png)

### 啟用專用資料端點
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218000612.png)

###### tags: `Azure` `Azure Container Registry` `Docker`

## 2.Key Vault 相關設定

### 準備憑證
文件裡面是用 openssl 產生自我簽署的憑證，但是如果這個 Registry 是要公開給其他人使用的，最好是正式發行(用錢買來的)的憑證。 

再來是需要準備2個以上的憑證, 1個給 REST API 用, 其他的給資料端點(實體檔案存放的區域)用。
舉例來說, 假設我希望我的Registry 叫`reg.sample.com`，就要產生`reg.sample.com`的憑證。

另外，如果資料端點在Azure日本東部的區域，就還要準備一個 `esjp-reg.sample.com` (名字沒有固定格式可以辦識就好)的憑證。如果還要把資料複寫到其他區域就要再準備其他的憑證。
  > 其實可以準備1個 *.sample.com 的憑證就都能用了
  


還有每個憑證的格式需要轉成PEM或PFX才能上傳到 Azure 到key Vault。 PEM的內容要包含private key跟public key。
```
-----BEGIN PRIVATE KEY-----  
.....  
-----END PRIVATE KEY-----  
-----BEGIN CERTIFICATE-----  
.....  
-----END CERTIFICATE-----
```

### 如果要使用自我簽署的憑證
可以在[這裡下載](https://slproweb.com/products/Win32OpenSSL.html)windows版openssl工具來產生憑證

- 產生openssl config 檔
```
[req]
default_bits = 4096
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=TW
CN=MyName
OU=sample.com
emailAddress=someone@sample.com
```
把上面的內容修改後存成openssl.cnf

- 建立自我簽署憑證, 產生public 憑證 跟 private key
```powershell
#這個指令會產生2個檔案
openssl req -config openssl.cnf -nodes -x509 -newkey rsa:4096 -keyout reg.sample.com.key.pem -out reg.sample.com.cert.pem -days 3650 -subj '/CN=reg.sample.com/O=sample./C=TW'
```
- 把上一步驟產生的公開憑證跟私鑰合併成同一個檔案

使用CMD
```command
 type .\reg.sample.com.cert.pem .\reg.sample.com.key.pem > reg-sample-com-pem.pem
```
或使用 powershell
```powershell
  Get-Content .\reg.sample.com.key.pem, .\reg.sample.com.cert.pem | Set-Content reg-sample-com-pem.pem
```

P.S 不要使用原本文件裡的cat方法 , cat產生出來的檔案編碼是 UTF-16 ,上傳到Key Vault會有問題

### 如果是使用正式發行的憑證
準備好你的憑證檔,請參考這篇 [匯入公開發行的憑証到 Azure Key Vault](https://hackmd.io/@ericstudio/匯入公開發行的憑証到AzureKeyVault)

### 上傳憑證到Azure Key Vault
1. 到 KeyVault> 憑證 > 產生/匯入
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210217234606.png)

2. 憑證建立方法 選擇 匯入, 設定名稱及上傳憑證檔後, 然後按下建立
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210217234619.png)

3. 建立完成後，進到憑證的內容裡，把祕密識別碼 copy 起來，等等會用到
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210217234629.png)

### 設定Key Vault 存取原則
1. 到 KeyVault> 存取原則 > 新增存取原則
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218002152.png)

2. 在 秘密權限 勾選 取得
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218002202.png)

3. 選取主體 > 選擇要授權的ACR ,(我的ACR叫SaplmeComReg)
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218002215.png)

4. 按下新增

![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218002239.png)

## 3.設定你的DNS

### 建立CNAME 記錄

- 以上面舉的例子，就是把登入domain name `reg.sample.com` 轉到 `samplecomreg.azurecr.io`
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218003640.png)

- 及資料domain name `esjp-reg.sample.com` 轉到 `samplecomreg.japaneast.data.azurecr.io`
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu.github.io/images/20210218003658.png)

## 4.呼叫微軟support
把登入domain的名字及其憑證的祕密識別碼，還有資料domain的名字及其憑證的祕密識別碼
都一起傳給微軟

範例:
```
Custom registry domain: reg.sample.com
key vault secret ID: https://mysamplekeyvault.vault.azure.net/secrets/sample-com/aa5dc6abc90b41538d7d9b8bb2072b7e

Custom data domain: esjp-reg.sample.com
key vault secret ID: https://mysamplekeyvault.vault.azure.net/secrets/esjp-sample-com/XXXXXXXXXXXXXXXXXXXX
```

## 5.如何在docker中使用自訂網域

這個時候，使用自我簽署的憑證或者買的憑證的差別就出來了。

### 正式憑證

如果是買的憑證，就直接使用一般的login指令輸入完帳密就可以了。
```
docker login reg.sample.com
```


### 自我簽署憑證
如果是使用自我簽署的憑證就還有額外的步驟，在[Docker官方文件](https://docs.docker.com/engine/security/certificates/#understand-the-configuration)裡有提到

#### 方法一
在 /etc/docker/certs.d/ (windows 10環境是C:\ProgramData\Docker\cert.d\)
建立一個domain名稱的目錄，以我們的例子目錄的名稱是`reg.sample.com`。

再把我們在之前產生的 reg-sample-com-pem.pem 改名成 ca.crt 之後放到剛才建立的目錄裡。

再來docker使用的login指令要多個--tls

```
docker login --tls reg.sample.com
```

#### 方法二
直接用 --tlscacert 指定憑証所在的位置

```
docker --tlscacert c:\mycert\reg-sample-com-pem.pem login reg.sample.com
```