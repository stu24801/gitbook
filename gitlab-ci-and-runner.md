# Gitlab CI & Runner

在Docker 中啟動Runner ，並註冊給對應的gitlab須注意使用的excutor類型，如果是docker 記得要先安裝docker.io在該容器中或者使用官方使用建立Runner方式將主機啟動Runner的Docker一併提供給Runner使用(此方式要注意提前於本機下載好要使用的image)。

```
docker run -d --name gitlab-runner --restart always \
    //提供主機Docker給Runner容器使用
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v gitlab-runner-config:/etc/gitlab-runner \
    gitlab/gitlab-runner:latest
```

{% embed url="https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/" %}
Docker 引擎共用
{% endembed %}

下方為啟動註冊的語法範例：

```docker
FROM gitlab/gitlab-runner:latest
LABEL authors="alex"
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 SERVER="XXXXX.com.tw" PORT=443
ENV CERTIFICATE="/etc/gitlab-runner/certs/${SERVER}.crt"
VOLUME ["/run/host-services/docker.proxy.sock", "gitlab-runner-config"]
# 設定憑證位置
RUN mkdir -p $(dirname "$CERTIFICATE")
# 匯入憑證
RUN openssl s_client -connect ${SERVER}:${PORT} -showcerts </dev/null 2>/dev/null | sed -e '/-----BEGIN/,/-----END/!d' | tee "$CERTIFICATE">/dev/null
# 匯入憑證
RUN openssl s_client -connect ${SERVER}:${PORT} -showcerts </dev/null 2>/dev/null | sed -e '/-----BEGIN/,/-----END/!d' | tee "$CERTIFICATE">/dev/null
# Install Docker
RUN apt-get update && \
    apt-get install -y docker.io && \
    rm -rf /var/lib/apt/lists/*
# 啟動Runner
RUN gitlab-runner register \
  --non-interactive \
  --url "https://xxxxx.com.tw/" \
  --registration-token "OOOXXXXOXOXOXOXOXO" \
  --executor "docker" \
  --docker-image container-register-t.ddddd.com.tw/dddddd/jenkins-slave-maven:v4.10.0.5\
  --description "docker-runner" \
  --maintenance-note "Free-form maintainer notes about this runner" \
  --tag-list "localhost,docker" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected" \
# 使用本機image,不使用Dockhub
  --docker-pull-policy="never"
```

{% embed url="https://docs.gitlab.com/runner/executors/#selecting-the-executor" %}
官方建議Excute類型
{% endembed %}

安裝docker.io的指令：

```
apt-get update && \
    apt-get install -y docker.io && \
    rm -rf /var/lib/apt/lists/*
```

{% hint style="info" %}
安裝前記得先將[DockDesktop. proxies 關閉](mac-docker-she-ding.md)，避免container 存取不到網路。
{% endhint %}
