# adb找不到设备

每次想用真机调试时，都遇到adb找不到设备的问题，实在太坑了，特此记录一下。

手机型号为 红米Note LTE，操作系统版本为Android4.4.4，MIUI7。

# 打开USB调试

1. Settings->About phone，连击MIUI version5次，可以打开开发者模式。
2. Additional settings->Developer options打开开发者模式设置界面，打开USB Debgging即可在adb中检测到设备了。

# 其他注意

有时候安装一个较大的apk，使用adb install之后，手机上会弹出是否允许安装，点击允许后界面突然消失了，实际上apk正在后台安装，请南信等待。
