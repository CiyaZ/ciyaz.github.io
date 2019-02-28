## 问题

AndroidStudio中，右键项目添加aidl文件，但是却不识别，Java的Service类中无法引用这个aidl文件。

## 解决

gradle配置android块里添加：
```java
sourceSets {
	main {
		aidl.srcDirs = ['src/main/aidl']
    }
}
```

然后rebuild。

## 原因

gradle只认`src/main/java`，右键添加的aidl文件在`src/main/aidl`，需要把这个路径添加进去。
