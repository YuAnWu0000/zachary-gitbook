# Cypress ─ 一款超強大的 E2E test framework

### 安裝

雖然 cypress 框架並不算小，但理論上 npm 一行指令便可以完成安裝。

```
npm install cypress --save-dev
```

> 為什麼我說理論上一行完成安裝呢？因為理想很豐滿，現實很骨感 QQ

實際上我在公司內網安裝時遇到了不少問題，例如：

##### 防火牆問題

大部分的公司內網應該都有鎖防火牆，比較常使用的 `registry.npmjs.org` 想必早已經加入白名單了，但 cypress 不單單只是個 npm 套件這麼簡單，作為一套測試框架，他有自己的 app 需要下載 (這樣才有酷酷的 Web UI 可以使用)。

這就導致了你在執行 `npm install cypress --save-dev` 時，需要額外開通 `https://download.cypress.io` 的防火牆。

然而，你會發現狀況依然沒有改善。

原因是`https://download.cypress.io`會重導向至 `https://cdn.cypress.io`，因此你必須同時開通這兩個才行。

##### CA 憑證問題

如果你們公司有針對防火牆設定憑證，要注意 cypress 從版本 11 之後加上了對 CA 的認證，所有未經授權的 CA 都會被系統駁回。(詳情請見 [change log](https://docs.cypress.io/app/references/changelog#11-0-0))

> Communication with the Cypress Dashboard will now verify CAs and reject any unauthorized calls. If you use a self-signed CA you will need to set npm_config_ca, npm_config_cafile, or NODE_EXTRA_CA_CERTS. Addresses #23980.

因此，當你執行`npm i cypress`時，會跳出 `Error: self signed certificate in certificate chain` 而無法成功下載。
