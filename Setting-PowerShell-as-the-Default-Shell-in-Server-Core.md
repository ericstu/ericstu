# 把 Windows Server Core 預設Shell改為Powershell

Windows Server Core 版本，預設是使用CMD當作shell ，當我們進到server core時，第一件事就是執行powershell，久了之後就會想把預設shell換成powershell，省去這一個工。

使用powershell更改Registry
```powershell
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name Shell -Value 'PowerShell.exe -NoExit'
```

改完重新啟動就生效了。

**但是**，預設的啟動目錄是 C:\Windows\System32 ，看了又很礙眼，所以想要把預設目錄改掉。

底下會使用建立一個新的設定檔的方式來設定預設目錄


先新建立一個profile檔
```powershell
New-Item -path $profile -type file –force
```

再來利用記事本編輯profile
```powershell
Notepad $profile
```

在裡面加上一行 Set-location 你要設的目錄
```
Set-location c:\
```

然後存檔就可以了

![](https://i.imgur.com/0Yt6exy.png)

