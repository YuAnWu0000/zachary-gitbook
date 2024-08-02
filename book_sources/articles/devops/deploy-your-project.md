# 搞定 Docker 跟 Nginx，輕鬆部署你的專案

「诶我把後端服務部署到 AWS 機器上囉！你看要不要也把你的前端專案部署上去。」<br>
「哦...好！可是我完全沒碰過 Docker 跟 Nginx 耶！」<br>
「......」<br>
後端對我比了一個讚後就回頭做他自己的事了。<br>
「這間公司不能待了 QQ」我心想。<br>

以上劇情純屬虛構，如有雷同，那就雷同。<br>

> 誰說只有後端可以兼任 DevOps，前端也可以打開 ubuntu terminal key 帥帥的 command 把自己的前端專案部署上機器上！成為八點檔裡面的駭客 🎉！

### 在這個前後端分離成為主流的時代，前端懂點 DevOps 的基礎會對公司省錢很有幫助 (X

### 在專案內新增 docker-compose.yaml

```
version: '3'
services:
  web:
    build:
      context: .
    environment:
      - 'API_HOST=X.X.X.X'
      - 'API_PORT=3001'
    ports:
      - 3000:80
```

### 在專案內新增 Dockerfile

```
FROM node:20 as build
WORKDIR /app
ADD package.json /app/
ADD package-lock.json /app/
RUN npm install

COPY . /app/
RUN npm run build
```
