# 我的服務還好嗎？監控容器的小撇步

上一篇教各位如何部署了自己的專案，身為前端工程師，平時網頁使用沒問題自然不會特別去監控容器狀態。<br>
不過一旦網頁出現問題，進入探索問題根源的過程，你有可能會需要以下指令...

### docker ps -a

- `-a`: 預設情況下，`docker ps` 只會顯示正在運行中的容器，`-a` 等同於 `--all`。
- `--format table`: 此行是預設不需要特別加，但在某些特殊情況下，你也可以變更為`--format json`。

| CONTAINER ID | IMAGE      | COMMAND                 | CREATED       | STATUS       | PORTS                     | NAMES        |
| :----------- | :--------- | :---------------------- | :------------ | :----------- | :------------------------ | :----------- |
| dcecfc18ffbb | my-project | "/docker-entrypoint..." | 3 minutes ago | Up 3 minutes | 0.0.0.0:10000->80/tcp,... | my-project-1 |

除了使用 `docker compose up --build -d` 重 build 以外，也可以簡單透過:

- `docker stop my-project-1`
- `docker start my-project-1`

來暫停或重啟 container，觀察你的服務行為。

### docker exec -it [container-name] bash

- `-i` (interactive): 保持標準輸入 (STDIN) 開啟，讓你跟容器互動。
- `-t` (tty): 分配一個 terminal 給該容器。

有時候你會想進 container 看看 nginx 或其他設定檔是否正確，這時候就可以使用這個指令。<br>
要離開容器輸入 `exit` 即可。

### docker logs -f [container-name]

- `-f`: 即時監控。

這個指令可以查看該 container 的 log，如果有在 container 當中使用 nginx，nginx 預設會將 access_log 的內容導出。<br>

主要用來確認 request 是否有正確導入、來源 IP、裝置為何等。<br>
當然，如果你的 container 一直啟動不了，也可以藉由 log 來查看錯誤訊息。<br>

如果嫌 log 太多，可以只印出倒數 50 行，並用時間排序:

```
docker logs --tail 50 --follow --timestamps [container-name]
```

> ### 不過如果你有用反向代理，nginx 如何導流的 log 預設是不會顯示在這裡的，需要額外編輯 access_log 的格式。

例如下面這樣：<br>

```
# nginx.conf
log_format proxy_log '[$time_local] $remote_addr - $remote_user "$host$request_uri" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"'
```
