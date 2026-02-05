# Maven筆記

## 啟動後觸發憑證問題需在VM arguments加入以下設定參數

-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true${archives\_current\_project}
