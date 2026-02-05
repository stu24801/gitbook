---
description: 常見錯誤
---

# Maven Pom 常見錯誤

ＩＤＥ中找不到對應套件，如：Plugin 'maven-resources-plugin:3.1.0' not found

１.原因為使用錯誤的Maven，使用到ＩＤＥ自帶的。

解決方案：

多添加\<groupId>，如下：

```
// Some code
<groupId>org.apache.maven.plugins</groupId> <----添加的
<artifactId>maven-resources-plugin</artifactId>
<version>3.1.0</version>
```

2.原因為ＩＤＥ找不到對應版本。

解決方案：

添加\<version>，如下：

```
// Some code
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
<version>${project.parent.version}</version> <----添加的
```
