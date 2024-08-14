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

_-x509: 指定要生成自簽憑證。_
_-nodes: 不加密私鑰。_
_-days 365: 設定憑證的有效期為 365 天。_
