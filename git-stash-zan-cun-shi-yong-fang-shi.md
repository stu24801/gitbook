# Git Stash（暫存）使用方式

如果遇到手上的作業尚未完成，又不想commit 到遠端分支，就可以進行Stash&#x20;

使用說明如下：[https://gitbook.tw/chapters/faq/stash](https://gitbook.tw/chapters/faq/stash)

1.先將異動內中放入local staged change

2.右鍵選擇stash，並輸入該暫存的說明訊息

3.更新修改你要先調整或同步的內容

4.若要取回暫存內容，則透過vs code開啟Stashes 頁籤，使用apply stash，將暫存導入到目前local的資料夾中，並顯示其中的衝突。
