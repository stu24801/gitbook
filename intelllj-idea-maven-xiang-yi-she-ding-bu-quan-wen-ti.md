# Intelllj idea maven 相依設定不全問題



解決方案：

**第一步**

然後設置 **VM Options**

`Setting -> Build,Execution... -> Build Tools -> Maven -> Runner`

設置

```
-Dmaven.wagon.http.ssl.insecure=true
-Dmaven.wagon.http.ssl.allowall=true
-Dmaven.wagon.http.ssl.ignore.validity.dates=true
```

**第二步**

1. 點擊右邊的Maven,點擊小寫m
2. 輸入命令運行

`mvn -U idea:idea`

1. 都下載完成後再點擊 **刷新按鈕** 即Reimport
