---
layout: post
---

```
title_game := "Adobe Flash Player 21"
title_ahk := "Robot"

;脚本开关
is_running := false

;单次循环间隔
pd := 0

;总循环次数
tt := 1

;鼠标X坐标
mx := 0

;鼠标Y坐标
my := 0

;启动脚本
^Numpad0::main()

;停止脚本
^Numpad1::
    is_running := false
    updateStatusBar_1()
return

;帮助
^NumpadDot::help()

;记录鼠标坐标
^Numpad9::pos()

main() {
    global

    if (!isWindowExist(title_game)) {
        return
    }

    if (is_running) {
        is_running := false
        return
    }

    is_running := true

    updateStatusBar_1()

    saveConfig()
    loop()
}

loop() {
    global

    mark := 0

    CoordMode, Mouse, Client
    while (is_running && (mark++ < tt || 0 = tt) && isWindowExist(title_game)) {
        Click %mx%, %my%
        updateStatusBar_2(mark)
        Sleep, pd
    }

    is_running := false
    updateStatusBar_1()
}

pos() {
    global

    if (!isWindowExist(title_game)) {
        return
    }

    if (is_running) {
        return
    }

    CoordMode, Mouse, Client
    MouseGetPos, mx, my

    SB_SetText("x: " . mx . ", y: " . my, 3)
}

help() {
    global

    if (isWindowExist(title_ahk)) {
        return
    }

    if (is_running) {
        return
    }

    Gui, Add, Text, section, 窗口标题:
    Gui, Add, Text, , 循环间隔:
    Gui, Add, Text, , 循环次数:
    Gui, Add, Text, , 快捷键:

    Gui, Add, Edit, w256 ys vtitle_game, %title_game%
    Gui, Add, Edit, w256 number vpd, %pd%
    Gui, Add, Edit, w256 number vtt, %tt%
    Gui, Add, Text, w256, Ctrl + 0 启动脚本`nCtrl + 1 停止脚本`nCtrl + 9 记录鼠标坐标

    Gui, Add, StatusBar
    SB_SetParts(64, 128)
    updateStatusBar_1()
    updateStatusBar_2(0)

    Gui, Show, x0 y0, %title_ahk%
}

GuiClose() {
    saveConfig()
    Gui, Destroy
}

isWindowExist(title) {
    global

    IfWinExist, %title%
    {
        WinActivate
        return true
    }

    return false
}

saveConfig() {
    Gui, Submit, NoHide
}

updateStatusBar_1() {
    global

    if (is_running) {
        SB_SetText("Running", 1)
    } else {
        SB_SetText("Stopped", 1)
    }
}

updateStatusBar_2(str) {
    SB_SetText("Loop:" . str, 2)
}
```