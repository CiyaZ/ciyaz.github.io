# 发布包到Nexus私服

注：有关Nexus的内容具体见`软件开发相关工具/Nexus-私有包管理仓库/Maven仓库类型`，阅读本篇前应先复习Nexus部署相关知识。

发布软件包时，需要配置两个地方：

1. Nexus私服的访问权限
2. 发布到哪个仓库

~/.m2/settings.xml
```xml
<server>
    <id>nexus-releases</id>
    <username>admin</username>
    <password>admin123</password>
  </server>
<server>
    <id>nexus-snapshots</id>
    <username>admin</username>
    <password>admin123</password>
  </server>
</servers>
```

pom.xml
```xml
<distributionManagement>
  <repository>
    <id>nexus-releases</id>
    <name>Nexus Release Repository</name>
    <url>http://192.168.43.164:8090/nexus/content/repositories/releases/</url>
  </repository>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <name>Nexus snapshots Repository</name>
    <url>http://192.168.43.164:8090/nexus/content/repositories/snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```
