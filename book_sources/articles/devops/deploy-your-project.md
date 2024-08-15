# 搞定 Docker 跟 Nginx，輕鬆部署你的專案

「诶我把後端服務部署到 AWS 機器上囉！你看要不要也把你的前端專案部署上去。」<br>
「哦...好！可是我完全沒碰過 Docker 跟 Nginx 耶！」<br>
「......」<br>
後端對我比了一個讚 👍 就回頭做他自己的事了。<br>
「這間公司不能待了」我心想。<br>

以上劇情純屬虛構，如有雷同，那就雷同。<br>

> 誰說只有後端可以兼任 DevOps，前端也可以打開 ubuntu terminal key 帥帥的 command 把自己的前端專案部署上機器上！成為八點檔裡面的駭客 🎉！

### 在這個前後端分離成為主流的時代，前端懂點 DevOps 的基礎會對公司省錢很有幫助 (X

> 那就讓我們開始吧！

##### 1. 在專案內新增 default.conf.template (nginx.conf)

**你可以簡單把 nginx 理解成一台本地的 web server，編輯他的設定檔可以幫你 host 靜態網頁在特定的 port 或者設定 proxy。**<br>

例如下面的設定就是去監聽 http 預設的 80 port，並且藉由比對 /api 的字串去做 rewrite，讓你的前端網頁在 call API 時可以直接用相對地址如: /api/profile，nginx 會直接作為 proxy 幫你把 /api/profile 轉到真正 server 的位置: http://x.x.x.x:port/profile

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
    proxy_pass http://api/ws;
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

額外加上 /ws 的 matching (此路徑可自行決定)，然後設定 http upgrade 相關的 header 就可以了！

##### 2. 在專案內新增 Dockerfile

試想如果一台機器上要同時運行 node 10, node 20, python2, python3 的專案，並且各自有完全獨立的 nginx 設定要怎麼做？<br>
這就是 docker 想要解決的問題了。<br>

**Docker 可以幫你建立出獨立的執行環境，如此一來在一台機器上你可以同時運行多個專案而不用擔心互相干擾，這也就是所謂容器化的概念 (可以想像他就是輕量化的 VM💻)。**<br>

要能建立容器需要你撰寫 Dockerfile 告訴他這個容器需要什麼東西，以下面為例，我使用官方提供 Nodejs version 20 的 docker image 來建立我專案的執行環境。

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

後面其實就是一步一步的把當前專案複製進去容器內，然後再用我們前面編輯過的 `default.conf.template` 來設定 nginx，並且 host build 完的靜態檔案在容器對外開放的 80 port。

##### 3. 在專案內新增 docker-compose.yaml

接下來你要做的就是把這個 docker build 好並且 run 起來。<br>
一個方式是你可以透過 `docker build` , `docker run` 這兩個 command 來做。<br>

或是你也可以跟我一樣不想打 command，那就可以用 docker compose 來幫你達成，好處是你不用每次都去記上面的 command 要怎麼下，只需要在 `docker-compose.yaml` 裡面定義好要做的事就行了，而且這個檔案可以上到 git 讓你享有版控的好處 👏。

_3.1 `build: context .` 他會幫你在當前目錄下找尋 Dockerfile 並且運行 docker build。_<br>
_3.2 `environment:` 對應到前面 `default.conf.template` 用到的環境變數 `${API_HOST}`, `${API_PORT}`。_<br>
_3.3 `ports: 3000:80` 他會幫你映射容器的 80 port 給外面機器的 localhost:3000。_<br>

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

##### 4. 最後 Command Line 執行 `docker compose up --build -d` 就搞定了 💪

_4.1 `docker compose up`: 根據 `docker-compose.yaml` 的設定來運行裡面所有的容器。_<br>
_4.2 `--build`: 啟動之前強制重 build 一次所有的 docker image，避免你使用到舊的。_<br>
_4.3 `-d`: 背景執行，相當於 `--detach`。_<br>

執行完以後你就可以下 `docker ps -a` 看看你的容器運行狀況了。

最後實際打開網站看看吧！到這一步你應該已經成功部署了，**不讓後端專美於前，身為前端也可以兼任 DevOps！🎉**
