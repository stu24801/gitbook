---
description: BDD TDD
---

# Cucumber in Eclipse

該測試模組必須運行java 1.8 +&#x20;

{% embed url="http://makeseleniumeasy.com/2020/01/17/cucumber-tutorial-1-setup-a-basic-maven-cucumber-4-junit-project-in-eclipse/" %}
該文章內有詳細的串接說明。
{% endembed %}

## 1.安裝eclipse Cucumber 插件

{% embed url="https://github.com/cucumber/cucumber-eclipse/releases/tag/0.0.23" %}

下載好Zip,透過Eclipse help > install new software > Add > Archive > 選擇剛載好的zip>下一步直到安裝完成。

## 2.添加Maven pom相依

```
  <dependency>
	    <groupId>io.cucumber</groupId>
	    <artifactId>cucumber-java8</artifactId>
	    <version>5.6.0</version>
	    <scope>test</scope>
	</dependency>
   	<dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-junit</artifactId>
        <version>5.6.0</version>
        <scope>test</scope>
    </dependency>
```

## 3.若出現找不到@Given等annotation&#x20;

請加入以下pom

```
     <dependency>
        <groupId>info.cukes</groupId>
        <artifactId>cucumber-core</artifactId>
        <version>1.2.5</version>
    </dependency>
    <dependency>
        <groupId>info.cukes</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>1.2.5</version>
    </dependency>
```

##

{% embed url="https://www.onejar99.com/bdd-cucumber-java-hello-example/" %}
撰寫BDD測試案例教學
{% endembed %}



