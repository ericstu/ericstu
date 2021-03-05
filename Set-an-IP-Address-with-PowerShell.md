---
GA: UA-139154617-3
disqus: ericstudio
langs: zh-tw
---
# 使用powershell設定IP

今天想要修改Server Core VM的IP，但是使用sconfig卻看不到我想要修改的網卡。


```
Microsoft (R) Windows Script Host Version 5.812
Copyright (C) Microsoft Corp. 1996-2006, 著作權所有，並保留一切權利

正在檢查系統...


===============================================================================
                         伺服器設定
===============================================================================

1) 網域/工作群組:               工作群組:  WORKGROUP
2) 電腦名稱:                    SWARMNODE1
3) 新增本機系統管理員
4) 設定遠端管理                 已啟用

5) Windows Update 設定:         手動
6) 下載並安裝更新
7) 遠端桌面:                    已停用

8) 網路設定
9) 日期和時間
10) 遙測設定未知
11) Windows 啟用

12) 登出使用者
13) 重新啟動伺服器
14) 關閉伺服器
15) 結束並返回命令列

輸入數字即可選取選項: 8


--------------------------------
           網路設定
--------------------------------


可用的網路介面卡

索引#           IP 位址         描述

  3             172.18.80.1     Hyper-V Virtual Ethernet Adapter

選取網路介面卡索引# (空白=取消):
```

但是用Powershell看是可以看到3張網卡的，我在想應該是在sconfig看不到虛擬的網卡。

```
PS C:\> Get-NetAdapter -Name *

Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
vEthernet (nat)          Hyper-V Virtual Ethernet Adapter             16 Up           00-15-5D-D1-B4-AB        10 Gbps
乙太網路                  Microsoft Hyper-V Network Adapter             8 Up           00-15-5D-2B-DD-04        10 Gbps
vEthernet (乙太網路)      Hyper-V Virtual Ethernet Adapter #2           5 Up           00-15-5D-2B-DD-04        10 Gbps
```

還好萬能的Powershell可以直接修改(其實也沒那麼直接)

```
#查看IP,從上個指令(Get-NetAdapter)可以看到 InterfaceIndex 就是ifIndex這個欄位
Get-NetIPAddress -InterfaceIndex 5
#移除IP
Remove-NetIPAddress -InterfaceIndex 5
#設定IP及遮罩
New-NetIPAddress -InterfaceIndex 5 -IPAddress 192.168.100.2 -PrefixLength 24
#移除gateway
Remote-NetRoute -InterfaceIndex 5
#設定gateway
New-NetRoute -InterfaceIndex 5 -NextHop 192.168.100.1 -DestinationPrefix 0.0.0.0/0
#測試
ping 192.168.100.1
ping 1.1.1.1
#查看DNS
Get-DnsClientServerAddress -Interface 5
#設定DNS
Set-DnsClientServerAddress -Interface 5 -ServerAddresses @("8.8.8.8","8.8.4.4")
#測試
ping www.google.com
```
###### tags: `powershell` `Server Core`