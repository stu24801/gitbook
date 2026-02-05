---
description: 設定SSL問題
---

# Git 設定 - VS code

1.初始化個人global 參數檔:&#x20;

\<git config --global user.name "USERNAME">

\<git config --global user.email XXXX@email.com>

{% embed url="https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-%E5%88%9D%E6%AC%A1%E8%A8%AD%E5%AE%9A-Git" %}
Config設定檔 與 相關Commend與法
{% endembed %}

2\. 於以下位置找到個人config檔&#x20;

\<C:\Users\\"USERNAME"\\.gitconfig>

3\. 於gitconfig檔案中加入以下設定

```
[http]
   sslBackend = schannel
   schannelCheckRevoke = false
[credential]
   helper = wincred
```

4\. 測試倉儲連線

\<git clone clone:https>&#x20;

5\. 初始化git 位置與檔案

移動到clone資料夾內，使用下列指令初始化

\<git init>

進行vscode git函式庫 位址更新語法

\<git remote add "位置名稱" "函式庫連結http">

6\. 重新啟動vscode 確認內容是否更新完成。

{% hint style="info" %}
登入帳號認證問題fatal: Authentication failed

注意平認證管理員是否有異動。

[https://dotblogs.com.tw/zeroade/2018/10/11/111941](https://dotblogs.com.tw/zeroade/2018/10/11/111941)
{% endhint %}
