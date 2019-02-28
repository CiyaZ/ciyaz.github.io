# 症状

开着Android Studio，开着AVD，调试OpenGLES，系统突然变卡，硬盘灯狂闪，图形界面失去响应。

# 解决

Linux下，ctrl+alt+F1-F7切换到其他控制台，kill掉所有和android相关的程序。

# 分析

可能是模拟器内存上的bug。具体原因不明。加大内存后已经很少出现了。
