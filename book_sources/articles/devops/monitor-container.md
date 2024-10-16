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
