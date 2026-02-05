# NodeJs/Linux 常用指令

## 改變NodeJs 執行目錄

//ONLY CMD 可用&#x20;

set PATH=D:\angulars\proxy\node-v14.0.0-win-x64;%PATH%;

## 設定npm proxy 若有內外網需求

npm config set registry https://OOOOOOO.XXXX.com.tw:port/OOOO/XXX

## 取消SSL連線驗證

npm config set strict-ssl false

## 設定curl proxy

```
設定你企業內的proxy url 並且關閉SSL確認
echo 'proxy = XXXXXX.com.tw:8080' >> ~/.curlrc
echo 'insecure' >> ~/.curlrc
```

## 移除Windows 資料目錄(含內部檔案資料)

rd /q /s D:\XXX\XXX&#x20;

## npm指令

如果執行過程中卡住，要思考是不是權限不夠需要sudo

安裝 npm install -->依照package.json 進行套件版本安裝

npm install --save -->安裝後更新本地端專案 package.json

npm install -g  -->安裝在全域使用的版本內

#### 安裝nvm &#x20;

用於管理多個npm版本&#x20;

&#x20;curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash&#x20;

查詢目前有安裝的版本

nvm ls

切換版本

nvm use \<version>

安裝特定版本npm

nvm install 18.17.0

測試連線Port號

lsof -i :3002

## linux指令

#### **切換帳號**

sudo su -  \<accountName>

#### **執行緒查詢**

ps -ef | grep XXXXX

觀看詳細文本

more 檔名





