# APP打包发布

这里我们介绍下如何在Android平台进行应用的打包发布，由于我没有苹果设备，IOS就没法演示了。

官方Android打包文档：[https://flutter.io/docs/deployment/android](https://flutter.io/docs/deployment/android)

## 秘钥文件

首先我们要准备好秘钥文件，这和打包Android原生代码相同。创建秘钥文件可以在普通Android工程中（注意Flutter工程中没有这个选项），点击`Build->Generate Signed APK`，然后在弹出的窗口中创建一个新的秘钥文件。

![](res/1.png)

## 创建key.properties

创建文件`<工程目录>/android/key.properties`，并填写以下内容：

```
storePassword=<你的密码>
keyPassword=<你的密码>
keyAlias=key0
storeFile=D:/workspace/document/android_key.jks
```

## 设置应用build.gradle

这个文件位于`<工程目录>/android/app/build.gradle`，注意不是外层的那个。

在`android {`前面加上：
```javascript
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```

替换：
```javascript
buildTypes {
    release {
        // TODO: Add your own signing config for the release build.
        // Signing with the debug keys for now, so `flutter run --release` works.
        signingConfig signingConfigs.debug
    }
}
```

为：
```javascript
signingConfigs {
    release {
        keyAlias keystoreProperties['keyAlias']
        keyPassword keystoreProperties['keyPassword']
        storeFile file(keystoreProperties['storeFile'])
        storePassword keystoreProperties['storePassword']
    }
}
buildTypes {
    release {
        signingConfig signingConfigs.release
    }
}
```

## 执行打包命令

进入应用根目录，执行`flutter build apk`。

生成的APK位于`<工程目录>/build/app/outputs/apk/release/app-release.apk`。

注：我发现模拟器里没法安装APK，报错`The APK failed to install. Error: Could not parse error string.`可能是模拟器的bug，我们可以用实机尝试，安装可以使用`adb`实现，同样的APK在我的`HUAWEI Mate10pro`上就是好使的。
