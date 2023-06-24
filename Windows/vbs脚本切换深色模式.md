适用于Quicker中切换的深色模式的代码

```vbscript
Set ws = CreateObject("Wscript.Shell")

'获取当前应用模式的注册表值
Value = ws.RegRead("HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\Personalize\AppsUseLightTheme")
path = "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Themes\Personalize\"


If (Value = 1) Then
'切换默认应用模式为浅色
t=ws.regwrite(path & "AppsUseLightTheme","0","REG_DWORD")
'切换默认Windows模式（"0"=深色，"1"=浅色）
t=ws.regwrite(path & "SystemUsesLightTheme","0","REG_DWORD")
'切换透明效果（"0"=不透明，"1"=透明）
t=ws.regwrite(path & "EnableTransparency","1","REG_DWORD")
Else
'切换默认应用模式为深色
t=ws.regwrite(path & "AppsUseLightTheme","1","REG_DWORD")
'切换默认Windows模式（"0"=深色，"1"=浅色）
t=ws.regwrite(path & "SystemUsesLightTheme","0","REG_DWORD")
'切换透明效果（"0"=不透明，"1"=透明）
t=ws.regwrite(path & "EnableTransparency","1","REG_DWORD")
End If

Wscript.Quit
```