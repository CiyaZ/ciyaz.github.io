# JRebel启动导致JVM崩溃

```
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  EXCEPTION_ACCESS_VIOLATION (0xc0000005) at pc=0x0000000000000000, pid=101696, tid=302192
#
# JRE version: Java(TM) SE Runtime Environment (8.0_31-b13) (build 1.8.0_31-b13)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.31-b07 mixed mode windows-amd64 compressed oops)
# Problematic frame:
# C  0x0000000000000000
#
# Failed to write core dump. Minidumps are not enabled by default on client versions of Windows
#
# If you would like to submit a bug report, please visit:
#   http://bugreport.java.com/bugreport/crash.jsp
#
...
```

这个问题比较少见，最终查明原因是因为使用了一个早期版本的JDK8，由于Oracle不为Windows提供非安装版的JDK，导致无法安装多个版本的JDK，所以从网上下载了一个广泛流传的别人安装好的JDK用作开发，但是这个版本太早了。该版本JVM可能有bug，使用JRebel就会崩溃，具体原因不明。
