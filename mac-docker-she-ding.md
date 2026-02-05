---
description: 說明如何安裝docker、公司harbor 設定
---

# MAC Docker 設定

安裝使用Desktop 版本，要設定環境參數：

`export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock`

`export DOCKER_HOST="unix://${HOME}/.colima/docker.sock"`

`export DOCKER_DEFAULT_PLATFORM=linux/amd64`

`export PATH=$PATH:/usr/sbin`

詳細安裝步驟：

{% embed url="https://medium.com/%E5%AE%B8-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98/mac-%E5%AE%89%E8%A3%9D-docker-%E5%8F%8A%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4-6a9cfaa55979" %}

安裝完畢後，請確認Desktop客戶端設定已經把proxies 開啟，這樣才可以讓container 可以連線到公司內部網站。

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

後續使用 docker pull 指令如果遇見下列權限問題：

<pre><code><strong>1. Cannot connect to the Docker daemon 
</strong><strong>2. Error response from daemon: unauthorized: unauthorized to access repository:
</strong></code></pre>

可以調整docker config 設定中的credsStore ，由desktop改成osxkeychain。

並透過下列指令進行：

```
//將權限設定給當前mac登入者
sudo chown -R $(id -u):$(id -g) $HOME/.docker
//更新credsStore 
sudo vi ~/.docker/config.json

```

詳細可參考下列連結：

{% embed url="https://forums.docker.com/t/error-failed-to-solve-error-getting-credentials-err-exit-status-1-out/136124/4" %}

如果遇到failed to verify certificate: x509: certificate signed by unknown authority 問題，代表該網站的憑證並沒有被信任，請進行下列處理：

```
// 設定環境參數
vi /etc/profile
export SERVER="XXXXXX.com.tw" 
export PORT=443 
export CERTIFICATE="/etc/gitlab-runner/certs/${SERVER}.crt"
source /etc/profile
// 建立憑證資料夾
mkdir -p $(dirname "$CERTIFICATE")
// 匯入憑證
openssl s_client -connect ${SERVER}:${PORT} -showcerts </dev/null 2>/dev/null | sed -e '/-----BEGIN/,/-----END/!d' | tee "$CERTIFICATE">/dev/null

```

