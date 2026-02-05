---
description: 常見問題彙整
---

# Junit Eclipse

## Class not found

在preference 下只要把Java——–compiler——-building——-Build path problems ——- incomplete build path 和 Circular dependencies改成warning。

## 若在Junit中無法找到mapstruct 實體化

參考[https://mapstruct.org/documentation/ide-support/](https://mapstruct.org/documentation/ide-support/)

Maven pom檔 需加入以下參數:

```
<properties>
  <!-- automatically run annotation processors within the incremental compilation -->
  <m2e.apt.activation>jdt_apt</m2e.apt.activation>
</properties>
```

在依序執行mvn clean package install > mvn test > eclipse junit&#x20;

