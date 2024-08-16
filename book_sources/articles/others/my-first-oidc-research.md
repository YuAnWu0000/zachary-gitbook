# 初探 OIDC ─ 揭開我們每天都在使用，卻不了解的黑盒子

多年前曾經串過 google Oauth2，現在想要把技術債一次補齊，因此這篇文誕生了！<br>
OIDC 是建構於 Oauth2 的一種"**身分驗證**"協議，常見的 Flow 有三種，本文將著重介紹我所使用的 Hybrid Flow。<br>
注意：請不要把 Oauth 與 OIDC 混為一談，**Oauth 是一種"授權"協議，沒有驗證功能**，`access_token`本身單純作為取得某權限的令牌使用，你沒辦法單純用 `access_token` 解析出他是誰。

### 關於身份定義那件事

- User: 一般使用者
- Relying Party (RP): 在這邊指的是 SPA 架構下的網站前端
- Identity Provider (IDP): 我這邊用的是 Authentik，最常見的有 Google, Facebook, Line...等第三方登入驗證服務。
- Web Backend: 也可以稱作 Resource Server。

### 1. 重導向的登入流程

<img src="../../images/my-first-oidc-research/login.PNG" width="600" >

1.1 使用者點擊登入<br>
1.2 前端將使用者導向 Authentik 的登入頁面<br>
