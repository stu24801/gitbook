# Nginx 使用手冊



{% embed url="https://blog.hellojcc.tw/nginx-beginner-tutorial/" %}

[https://kknews.cc/zh-tw/code/vyvy42a.html](https://kknews.cc/zh-tw/code/vyvy42a.html)

## 指令分享

netstat -tnlp      查詢Linux 上使用中的Port 號

systemctl status / start / stop rh-nginx114-nginx.service    啟動/狀態查詢/停止 nginx 伺服器

## 控制檔(nginx.conf)擺放位置

/opt/rh/rh-nginx114/register.content/etc/opt/rh/rh-nginx114/nginx/conf.d/xxxx.conf

## 控制檔範例

```
server {
    
    access_log /log/wss_access.log;
    error_log /log/wss_error.log;
    
    listen 8443 ssl default_server;
    server_name  wmtpoc01wt.test.com.tw;
    
    ssl on;
	  ssl_certificate /home/wispoc/wmtpoc01wt.test.com.tw.pem;  
	  ssl_certificate_key /home/wispoc/wmtpoc01wt.test.com.tw.key;
    
    location / {
      root   /home/wispoc/esb_wmt_frontend/;
      ###### 同源CORS解法 ######
       if ($http_origin ~* (https?://[^/]*\.domain\.com(:[0-9]+)?)$) {
            set $cors "true";
        }

        if ($request_method = 'OPTIONS') {
            set $cors "${cors}options";
        }

        if ($request_method = 'GET') {
            set $cors "${cors}get";
        }
        if ($request_method = 'POST') {
            set $cors "${cors}post";
        }

        if ($cors = "trueget") {
            add_header 'Access-Control-Allow-Origin' "$http_origin";
            add_header 'Access-Control-Allow-Credentials' 'true';
        }

        if ($cors = "truepost") {
            add_header 'Access-Control-Allow-Origin' "$http_origin";
            add_header 'Access-Control-Allow-Credentials' 'true';
        }

        if ($cors = "trueoptions") {
            add_header 'Access-Control-Allow-Origin' "$http_origin";
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,Keep-Alive,X-Requested-With,If-Modified-Since';
            add_header 'Content-Length' 0;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            return 204;
        }
        
      index index.html index.html;
    }
    location /esb_wmt_frontend {
      root   /home/wispoc/;
      #try_files $uri index.html;
      index index.html index.html;
    }
    location /header_component/ {
      root   /home/wispoc/;
	    index index.html index.html;
    }
    location /custmrInfo/ {
      root   /home/wispoc/;
	    index index.html index.html;
    }
    location /userRoleConfig/ {
      root   /home/wispoc/;
	    index index.html index.html;
    }
    
    location /esb_wmt_frontend/restfulApiForward/agent/ {
      proxy_pass http://wmtpoc01at.test.com.tw:8080/wise-ap/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
      proxy_max_temp_file_size 0;
    }
    location /wmt_frontend/appInit {
    # 定義 header 變數, 記錄使用者的 IP
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
      proxy_max_temp_file_size 0;
      proxy_pass http://wmtpoc01at.test.com.tw:8080/wise-ap/agent/appInit;
    }
    location /wmt_frontend/restfulApiForward/custmrInfo/ {
      proxy_pass http://wmtpoc01at.test.com.tw:9080/wmt-custmrInfo-ap/custmrInfo/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
      proxy_max_temp_file_size 0;
    }
}
```

## Reverse Proxy Server 設定規格

```
server { listen 80; server_name www.test.com;

# 情形A
# API查找 http://www.test.com/testa/aaaa
# 後端的request_uri為: /testa/aaaa
location ^~ /testa/ {
    proxy_pass http://127.0.0.1:8801;
}

# 情形B
# API查找 http://www.test.com/testb/bbbb
# 後端的request_uri為: /bbbb
location ^~ /testb/ {
    proxy_pass http://127.0.0.1:8801/;
}

# 情形C
# 下面這段location是正確的
location ~ /testc {
    proxy_pass http://127.0.0.1:8801;
}

# 情形D
# 下面這段location是錯誤的
#
# nginx -t 時，會報下列錯誤: 
#
# nginx: [emerg] "proxy_pass" cannot have URI part in location given by regular 
# expression, or inside named location, or inside "if" statement, or inside 
# "limit_except" block in /opt/app/nginx/conf/vhost/test.conf:17
# 
# 当location为正則表達式，proxy_pass 不能包含URI部分。本例中包含了"/"
location ~ /testd {
    proxy_pass http://127.0.0.1:8801/;   # 記住，location為正則表達式時，不能這樣寫
}

# 情形E
#  http://www.test.com/ccc/bbbb
# 後端的request_uri為: /aaa/ccc/bbbb
location /ccc/ {
    proxy_pass http://127.0.0.1:8801/aaa$request_uri;
}

# 情形F
# 查找 API http://www.test.com/namea/ddd
# 後端的request_uri為: /yongfu?namea=ddd
location /namea/ {
    rewrite    /namea/([^/]+) /yongfu?namea=$1 break;
    proxy_pass http://127.0.0.1:8801;
}

# 情形G
# 查找 API http://www.test.com/nameb/eee
# 後端的request_uri為: /yongfu?nameb=eee
location /nameb/ {
    rewrite    /nameb/([^/]+) /yongfu?nameb=$1 break;
    proxy_pass http://127.0.0.1:8801/;
}

access_log /data/logs/www/www.test.com.log;
}
```

##
