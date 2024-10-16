# 還在為不同執行環境寫 if else? 你需要的是.env

公司的前端專案需要三個執行環境，分別是 develpoment, uat, production，你費盡千辛萬苦將專案分別部署上三台機器以後，發現需要根據執行環境來判斷某變數 CLIENT_ID，這時候，你會怎麼做？

才一個變數而已，我分別在三台機器上面改就好了。
然後你發現，每次版本更新的時候你都要特別注意這個檔案，因為深怕 CLIENT_ID 被覆蓋掉。

注意到了嗎？你做了一個**擴展性很差**的決定，因為如果未來有更多變數需要依賴於執行環境，你就會需要關注更多檔案，不過當你有了 .env，這些額外心力是完全可以避免的。

### 那麼要如何根據環境引入不同的.env 呢？

如果你使用 CRA 建立專案，可以參考這個[文件](https://create-react-app.dev/docs/adding-custom-environment-variables/#what-other-env-files-can-be-used)。

> 如果不是...那你可能要稍微了解一下 Webpack DefinePlugin，大部分的環境變數套件都是靠它實作的。

簡單說，CRA 專案會在你執行指令時自動判斷要引入哪個對應的.env 檔 (權重由大到小):

- npm run start : .env.development.local, .env.local, .env.development, .env
- npm run build : .env.production.local, .env.local, .env.production, .env
- npm run test : .env.test.local, .env.test, .env(note .env.local is missing)

所以我們可以在專案內新增一個.env.development 檔案:

```
REACT_APP_CLIENT_ID = myLocalClientId
```

一個.env.production.檔案:
