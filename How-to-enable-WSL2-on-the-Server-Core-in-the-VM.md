# 在VM的Windows Server Core裡啟用WSL2

### 啟用VM的巢狀虛擬化

注意: Guest OS 的版本要在 Windows Server build 18945 版本以上，這裡的範例是使用 Windows Server 2004的版本

在VM安裝完成後，第一步就是先啟用VM的巢狀虛擬化

以Hyper-V為例(使用powershell)
```powershell
Get-VM "VM的名字" | Set-VMProcessor -ExposeVirtualizationExtensions $true
```
各種VM啟用的方式，參考資料如下

Hyper-V 參考[這裡](https://docs.microsoft.com/zh-tw/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)

VMware 參考 [這裡](https://communities.vmware.com/t5/Nested-Virtualization-Documents/Running-Nested-VMs/ta-p/2781466) 或 [這裡](https://ephrain.net/vmware-%E5%85%81%E8%A8%B1-esxi-%E4%B8%8A%E7%9A%84%E8%99%9B%E6%93%AC%E6%A9%9F%E5%99%A8%E9%96%8B%E5%95%9F-nested-vm-%E5%8A%9F%E8%83%BD/)

VirtualBox 參考[這裡](https://ostechnix.com/how-to-enable-nested-virtualization-in-virtualbox/)


### 進到VM裡安裝WSL2

#### 1.啟用 Microsoft-Windows-Subsystem-Linux 及 VirtualMachinePlatform 這2個功能

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
 安裝完之後重新開機 
 
```powershell
Restart-Computer
```

#### 2.下載 Linux 核心更新套件
```powershell
Invoke-WebRequest -Uri https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi -OutFile c:\wsl_update_x64.msi  -UseBasicParsing
```
下載完成後執行
```
c:\wsl_update_x64.msi
```
會有畫面出來讓你按下一步(我還以為Server Core都沒有UI咧)。

![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/kslNXcK.png)

安裝完成之後重新開機
```powershell
Restart-Computer
```

#### 3.把 WSL 預設版本改成 WSL2
```powershell
wsl --set-default-version  2
```
成功啟用會有下列訊息，正常只會出現一行 有關 WSL 2.......
如果出現其他的表示沒有啟用成功

```dos
PS C:\> wsl --set-default-version 2
有關 WSL 2 的主要差異詳細資訊，請瀏覽 https://aka.ms/wsl2
```

#### 4.到[這裡](https://docs.microsoft.com/en-us/windows/wsl/install-manual) 依個人喜好下載給WSL2 用的 Linux 發行版本

我是下載Ubuntu 20.04 這個版本 
```powershell
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile Ubuntu.zip -UseBasicParsing
```

#### 5.下載完成之後解壓縮
```powershell
Expand-Archive .\Ubuntu.zip .\Ubuntu
```

#### 6.到解壓縮的目錄執行安裝檔 ， 以Ubuntu 20.04為例是 ubuntu2004.exe

```powershell
PS C:\> cd .\Ubuntu\
PS C:\Ubuntu> .\ubuntu2004.exe
```
#### 7. 安裝完成之後需設定Linux 的帳號密碼

```dos
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: 
New password:
Retype new password:
passwd: password updated successfully
```

輸入exit 就會離開linux

#### 8.檢查設定是否正常

```powershell
PS C:\Ubuntu> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-20.04    Running         2
```


參考資料

1.[Enable the Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-on-server)

2.[Windows 10 上適用於 Linux 的 Windows 子系統安裝指南](https://docs.microsoft.com/zh-tw/windows/wsl/install-win10)

###### tags: `Windows Server 2004` `Windows Server Core` `WSL2`