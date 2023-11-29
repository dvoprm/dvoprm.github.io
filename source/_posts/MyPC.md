---
title: 我的生产力工具
categories: 未分类
date: 2023-11-29 15:54:56
updated: 2023-11-29 15:54:56
tags:
keywords:
description:
top_img:
---

目前我的主力PC为ROG枪神7-4080版本，2根32G英睿达4800频率内存条，加装了4TB便宜固态。本文重点是软件方面。

# 操作系统
电脑刚到手就被我重装系统，使用一段时间后发现更新为Windows 11 23h2后没有Copilot (preview) ，这让我强迫症的我忍无可忍（我可以不用，但是不可以没有），所以有第二次重装系统，使用一段时间后逐渐稳定，故有本文。

## 重装系统
选择 
- 区域：美国
- 语言：English (US)

联想到美区苹果账号功能最多，这里使用美国区域也是合情合理的吧。

很多国产软件的文字编码会遇到问题，所以需要进入系统之后做一些修改：

![Change system locale...](change_system_locale.png)

## 从macOS回来的Windows用户
macOS有很多讨喜的特性（至少对于习惯的人来说），下面给出一些尽量弥补的措施。

### 鼠标滚轮自然滚动！
Google搜索后根据相关措施设置即可。

### command+C command+V
再也不想因为在命令行/终端想复制而打断程序运行了！

考虑到Windows系统中还有另外的复制/粘贴快捷键，
- Ctrl Insert：复制
- Shift Insert：粘贴
- Shift Delete 剪切

那么我们只需要把Win+C、Win+V、Win+X**快捷键映射**过去就好了，这里使用[Microsoft PowerToys](https://learn.microsoft.com/en-us/windows/powertoys/)工具实现此功能，如下图所示：
![快捷键映射](key_remap.jpg)

### CAPSLOCK 中英文切换
如上图绿色内容所示，做一个按键的映射即可。

### fk微软拼音输入法！
我并不希望使用拼音输入法**输入英文**，因为我有英文输入法，但是偏偏微软拼音会自作主张地在某些场合（比如回到桌面、浏览器地址栏）切换成英文模式，试图教用户打字。

简单搜索下可在知乎[Windows 10 如何关闭微软拼音输入法的英文输入模式？](https://www.zhihu.com/question/461304943)找到措施，经个人试验，下面给出可用的解决方法：
- 安装[AutoHotkey](https://www.autohotkey.com/)
- 保存下面内容到新建文件 "文件名随便.ahk"，可以双击文件后享受下不被教打字的快感，但是如果要开机自动启动这个脚本的话，则需要下面这步↓
- 打开文件管理器，地址栏输入 shell:startup ，将"文件名随便.ahk"拷贝到该位置

```
#NoTrayIcon
#Include %A_ScriptDir%

timeInterval := 500

;    +-------------------------+-------------------------+
;    |     SubLanguage ID      |   Primary Language ID   |
;    +-------------------------+-------------------------+
;    15                    10  9                         0   bit

InChs(hWnd) {
    ThreadID:=DllCall("GetWindowThreadProcessId", "UInt", hWnd, "UInt", 0)
    ime_status := DllCall("GetKeyboardLayout", "int", ThreadID, "UInt")
    return (ime_status & 0xffff) = 0x804 ; LANGID(Chinese) = 0x804
}

SwitchImeState(id) {
  SendMessage(0x283,  ; WM_IME_CONTROL
              0x002,  ; wParam IMC_SETCONVERSIONMODE
              1025,   ; lParam (Chinese)
              ,       ; Control (Window)
              id)
}

DetectHiddenWindows True

outer:
Loop {
  try {
    hWnd := WinGetID("A")
    id := DllCall("imm32\ImmGetDefaultIMEWnd", "Uint", hWnd, "Uint")

    if (InChs(hWnd)) {
      SwitchImeState(id)
    }
  } catch as e {
    ; ^Esc 开始菜单弹窗，会卡死在找不到当前窗口
    continue("outer")
  }

  Sleep(timeInterval)
}
```
