# 搞定 Docker 跟 Nginx，輕鬆部署你的專案

「诶我把後端服務部署到 AWS 機器上囉！你看要不要也把你的前端專案部署上去。」<br>
「哦...好！可是我完全沒碰過 Docker 跟 Nginx 耶！」<br>
「......」<br>
後端對我比了一個讚後就回頭做他自己的事了。<br>
「這間公司不能待了 QQ」我心想。<br>

以上劇情純屬虛構，如有雷同，那就雷同。<br>

> 誰說只有後端可以兼任 DevOps，前端也可以打開 ubuntu terminal key 帥帥的 command 把自己的前端專案部署上機器上！成為八點檔裡面的駭客 🎉！

### 在這個前後端分離成為主流的時代，前端懂點 DevOps 的基礎會對公司省錢很有幫助 (X

##### 在專案內新增 Dockerfile

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

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

##### 在專案內新增 docker-compose.yaml

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
```

##### 在專案內新增 default.conf.template (nginx.conf)

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

###### 順帶一提，如果你的專案有用到 websocket 的話可以這樣寫...

```
map $http_upgrade $connection_upgrade {
  default Upgrade;
  '' close;
}
server {
  include   /etc/nginx/mime.types;
  default_type  application/octet-stream;
  root  /usr/share/nginx/html/;

  absolute_redirect off;

  listen 80;

  location = /ws {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP    $remote_addr;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_http_version 1.1;
    proxy_redirect off;
    proxy_buffering off;
    proxy_read_timeout 86400s;
    proxy_send_timeout 86400s;
  }

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
