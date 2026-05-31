для v2rayn, [[Замедление YouTube]]

```
#Requires AutoHotkey v2.0
#SingleInstance Force

; Порт локального proxy в v2rayN
ProxyServer := "127.0.0.1:10808"

RegKey := "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
ProxyOverride := "localhost;127.*;<local>"

; Неизвестная клавиша ноутбука — toggle system proxy
SC074_Down := false

$*SC074::
{
    global SC074_Down

    if SC074_Down
        return

    SC074_Down := true
    ToggleProxy()
}

$*SC074 Up::
{
    global SC074_Down
    SC074_Down := false
}

; Блокируем Win+H, чтобы не открывалось окно диктовки
#h::
{
    return
}

LWin & h::
{
    return
}

RWin & h::
{
    return
}

ToggleProxy() {
    global RegKey
    enabled := RegRead(RegKey, "ProxyEnable", 0)
    SetProxy(!enabled)
}

SetProxy(enable) {
    global RegKey, ProxyServer, ProxyOverride

    if enable {
        RegWrite 1, "REG_DWORD", RegKey, "ProxyEnable"
        RegWrite ProxyServer, "REG_SZ", RegKey, "ProxyServer"
        RegWrite ProxyOverride, "REG_SZ", RegKey, "ProxyOverride"
        RegWrite "", "REG_SZ", RegKey, "AutoConfigURL"

        RefreshSystemProxy()
        TrayTip "v2rayN proxy", "System proxy: ON → " ProxyServer, 1
    } else {
        RegWrite 0, "REG_DWORD", RegKey, "ProxyEnable"
        RegWrite "", "REG_SZ", RegKey, "ProxyServer"
        RegWrite "", "REG_SZ", RegKey, "ProxyOverride"
        RegWrite "", "REG_SZ", RegKey, "AutoConfigURL"

        RefreshSystemProxy()
        TrayTip "v2rayN proxy", "System proxy: OFF", 1
    }
}

RefreshSystemProxy() {
    ; INTERNET_OPTION_SETTINGS_CHANGED = 39
    ; INTERNET_OPTION_REFRESH = 37
    DllCall("wininet\InternetSetOptionW", "Ptr", 0, "UInt", 39, "Ptr", 0, "UInt", 0)
    DllCall("wininet\InternetSetOptionW", "Ptr", 0, "UInt", 37, "Ptr", 0, "UInt", 0)
}
```