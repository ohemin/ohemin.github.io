---
title: 解决文件已被macOS使用无法打开
date: 2018-10-11 13:40:00 +0800
categories: [Mac, tip]
---


在mac系统使用过程中有时会遇到想打开的文件无法打开的情况，这时候系统会提示 `已被macOS使用,不能打开`

按如下方式操作即可解决

1. 选中想要打开的文件，按一下copy快捷键 <kbd>Command + c</kbd>
2. 打开应用程序`终端` 有些系统版本名称为 `terminal`
3. 输入 `xattr -d com.apple.FinderInfo `，注意最后有一个空格
4. 保持输入焦点在`终端`窗口中，按下粘贴快捷键 <kbd>Command + v</kbd>，这里刚刚复制的文件路径会粘贴到窗口中
5. 按下 <kbd>Enter</kbd> 执行输入的命令

这样再去打开刚刚的文件，就能成功打开了