# 初探 OIDC ─ 揭開我們每天都在使用，卻不了解的黑盒子

我多年前曾經串過 Google Oauth2，現在想要把技術債一次補齊，因此這篇文誕生了！<br>
OIDC 是建構於 Oauth2 之上的一種"**身分驗證**"協議，常見的 Flow 有三種，本文將著重介紹我所使用的 Hybrid Flow。<br>

注意：請不要把 Oauth 與 OIDC 混為一談，**Oauth 是一種"授權"協議，沒有驗證功能**，`access_token` 本身只單純作為取得某權限的令牌使用，在 Oauth 中你沒辦法單純用 `access_token` 解析出他是誰。<br>

不過 OIDC 就不同了，他的 `access_token` 通常是 JWT，你可以透過 decode JWT 來獲得 payload，進而得知該名使用者的基本資訊。

### 關於身份定義那件事

在講述任何流程之前，我喜歡先把角色定義好，避免流程搞懂旦角色混淆而不知如何應用的情況。

- **User:** 一般使用者
- **Relying Party (RP):** 在這邊指的是 SPA 架構下的網站前端
- **Identity Provider (IDP):** 我這邊用的是 Authentik，常見的有 Google, Facebook, Line...等第三方登入驗證服務
- **Web Backend:** 也可以稱作 Resource Server

### 1. 重導向的登入流程

<img src="../../images/my-first-oidc-research/login.PNG" width="1000" >

_**1.1 使用者點擊登入**_<br>
_**1.2 前端隨機產生 state 並儲存在 sessionStorage 內**_<br>
_**1.3 前端將使用者導向 Authentik 的登入頁面，一併帶上 state, redirect_uri 等參數**_<br>
_**1.4 登入成功後 Authentik 根據 redirect_uri 導回前端的 /callback 頁面，並帶上 state, code 參數**_<br>

在這個階段，Authentik 成功得知了該使用者是誰，並且將他導回到我們的前端頁面，但此時網頁前端還沒有取得對應的 access_token，只有拿到 Authentik 回傳的 state 跟 code 而已。<br>

補充：在這邊也可以直接請 Authentik 把 access_token 帶在網址列上回傳給你，省略回傳 code 的步驟直接完成登入流程。這就是所謂的 **OIDC Implicit Flow，屬於流程相對簡單，但安全性相對較低的一種方式**。

### 2. 取得 access_token

<img src="../../images/my-first-oidc-research/getToken.PNG" width="1000" >

_**2.1 驗證 Authentik 帶回來的 state 跟當初儲存在 sessionStorage 是否相同**_<br>
_**2.2 若不同，則顯示錯誤給使用者**_<br>
_**2.3 若相同，則用 POST 方法打向 /token 並帶上 code 等參數**_ _(`Content-type: application/x-www-form-urlencoded`)_ <br>
_**2.4 Authentik 回傳 access_token, refresh_token**_<br>
_**2.5 前端將這兩個 token 儲存至 localStorage**_<br>
_**2.6 開始與 Web Backend 互動，開始一般的登入流程 (例如把 access_token 帶在 Authorization Header 裡面：`Authorization: Bearer ${access_token}`)**_<br>

### 3. Refresh Token 的流程

<img src="../../images/my-first-oidc-research/refreshToken.PNG" width="1000" >

_**3.1 前端可透過解析 access_token (JWT decode)來取得 expired time，主動設定一個 timer 到期通知**_<br>
_**3.2 收到 timer 通知後，用 POST 方法打向 /token 並帶上 refresh_token 等參數**_<br>
_**3.3 Authentik 回傳 access_token, refresh_token**_<br>
_**3.4 前端將這兩個新的 token 儲存至 localStorage**_<br>

### References

深入淺出 OpenID Connect (一): https://shuninjapan.medium.com/%E6%B7%B1%E5%85%A5%E6%B7%BA%E5%87%BA-openid-connect-%E4%B8%80-8701bbf00958<br>
2022 鐵人賽 Identity Management: https://ithelp.ithome.com.tw/articles/10300430
