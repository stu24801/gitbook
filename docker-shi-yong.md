# Docker 使用

注意在使用指令docker buildx builder image:tag 時，要注意其指令格式，最後一定要有空格與.號，且執行過程如果遇到認證授權timeout 就代表該docker權限沒有添加至使用者群組，導致每次執行都需要帶入sudo指令。

docker buildx build image:tag 透過同路徑下的Dockfile 來打包出對應的映像檔(需要sudo)

docker context ls 可以查看目前機器上所有已安裝的docker項目。

docker info 可以查看目前的啟動狀態。

colima start 可以啟動colima docker。

colima stop 可以停止colima docker。

colima status 可以確認狀態colima docker。

