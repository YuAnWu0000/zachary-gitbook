# 讓你的網站更安全，加上 SSL 憑證吧

後端：「我把我的後端加上 https 囉！你也幫你的前端加一下吧！」<br>
我：「蛤?!」<br>
後端：「你可以先產一個自簽憑證出來掛上去，我請網管去拿正式憑證。」<br>
我：「蛤......」<br>
我：「那是像員工證一樣掛在伺服器上就可以了嗎？」<br>

### 產出自簽 SSL 憑證

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./ssl/nginx-selfsigned.key -out ./ssl/nginx-selfsigned.crt
```

_-x509: 指定要生成自簽憑證。_<br>
_-nodes: 不加密私鑰。_<br>
_-days 365: 設定憑證的有效期為 365 天。_<br>
_-newkey rsa:2048: 建立一個新的 RSA 私鑰，長度為 2048 位，並使用該密鑰生成憑證。_<br>
_-keyout ./ssl/nginx-selfsigned.key: 指定私鑰儲存位置。_<br>
_-out ./ssl/nginx-selfsigned.crt: 指定憑證儲存位置。_<br>

現在，你應該有 private key & cert 兩個檔案了，下一步要想的是如何把它們"掛"上去 👆~

### 修改 nginx 設定，讓它監聽 https 的 443 port

> 還記得我們前一篇文章學到的 nginx 跟 docker 嗎？

// default.conf.template
我們之前只有監聽 http 預設的 80port

```
server {
  include   /etc/nginx/mime.types;
  default_type  application/octet-stream;
  root  /usr/share/nginx/html/;
  absolute_redirect off;
  listen 80;
  location ^~ /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://api;
  }
  location ^~ / {
    try_files $uri /index.html;
  }
}
upstream api {
  server ${API_HOST}:${API_PORT}
}
```

現在讓我們加上監聽 443 port 的部分：

```
server {
  include   /etc/nginx/mime.types;
  default_type  application/octet-stream;
  root  /usr/share/nginx/html/;
  absolute_redirect off;
  listen 80;
  location ^~ /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://api;
  }
  location ^~ / {
    try_files $uri /index.html;
  }
}
server {
  include   /etc/nginx/mime.types;
  default_type  application/octet-stream;
  root  /usr/share/nginx/html/;
  absolute_redirect off;
  listen 443 ssl;
  ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt
  ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key
  location ^~ /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://api;
  }
  location ^~ / {
    try_files $uri /index.html;
  }
}
upstream api {
  server ${API_HOST}:${API_PORT}
}
```

眼尖的讀者應該發現了，其實只有這三行不同：

```
listen 443 ssl;
ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt
ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key
```

**記得 `ssl` 一定要加喔！不然你依舊會 host 一個 http (血淋淋的親身經歷 😥)。**<br>
**路徑的部分因為我們專案是跑在 docker 裡面的，所以還要想辦法把這個內部路徑指到外面機器的./ssl/**<br>

那就讓我們來看看 docker 要怎麼改吧！

### 修改 Dockerfile

```
FROM node:20 as build
WORKDIR /app
ADD package.json /app/
ADD package-lock.json /app/
RUN npm install

COPY . /app/
RUN npm run build

FROM nginx:stable
COPY --from=build /app/build /usr/share/nginx/html
COPY default.conf.template /etc/nginx/templates/

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

只改了一行 `EXPOSE 80 443`，讓 docker 多把 443 port 開出來。

### 修改 docker-compose.yaml

```
version: '3'
services:
  web:
    build:
      context: .
    environment:
      - 'API_HOST=X.X.X.X'
      - 'API_PORT=XXXX'
    ports:
      - 3000:80
    volumes:
      - ./ssl:/etc/nginx/ssl
```

新增 `volumes: - ./ssl:/etc/nginx/ssl` 讓 docker 內部的 `/etc/nginx/ssl` 可以直接掛載到外面的 `./ssl`。
**也就是我們一開始用 openssl 產出來的私鑰跟憑證的存放位置。**

### 重跑 docker compose up --build -d 就大功告成了！

是不是比想像中的要簡單呢？<br>

如果要換成正式憑證只要把它丟進去前面設定好的路徑就可以了，這也是為什麼我採用掛載的方式。

注意：在用自簽憑證的時候
