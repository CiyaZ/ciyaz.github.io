# ADB是什么

ADB（安卓调试桥）是谷歌提供的安卓调试工具，通过在PC和Android设备中建立socket通信，达到在PC中开发调试，Android设备上运行的目的。ADB在Android SDK中提供，添加环境变量后即可使用。

# ADB常用命令

* `adb devices` :列出当前电脑所连接的android设备
* `adb push pc_path  phone_path` :将电脑端文件放到手机端
* `adb pull phone_paht pc_path` :将手机端文件拉到电脑端
* `adb install [-r] apkpath`：安装一个电脑端的apk文件。`-r`：强制安装
* `adb uninstall packagename`： 卸载一个应用

* `adb kill-server` : 结束adb服务的链接
* `adb start-server` ：开启adb服务的链接（任意一条adb指定都会自动执行`adb start-server`）

* `adb shell`：进入当前设备linux shell下
* `adb shell` + `logcat`：在PC终端打开Android设备的logcat
* `adb shell` + `exit`：退出Android linux shell

注意： 如果当前电脑链接的是多台android设备，需要指定操作的是哪台设备，需要在adb后加 `-s` 设备序列号。

# 常见问题

1. adb使用5037端口，端口占用无法启动，检查计算机的5037端口情况。
2. AndroidStudio的Logcat没有输出，可能是adb卡死了，或者AndroidStudio本身的bug，重启adb，再不行的重启AndroidStudio，一般就好使了。
