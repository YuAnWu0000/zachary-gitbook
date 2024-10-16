# 還在為不同執行環境寫 if else? 你需要的是.env

公司的前端專案需要三個執行環境，分別是 develpoment, uat, production，當你費盡千辛萬苦將專案分別部署上三台機器以後，發現需要根據執行環境來判斷 CLIENT_ID，這時候，你會怎麼做？

才一個變數而已，我分別在三台機器上面改就好了。
然後你發現，每次版本更新的時候你都要特別注意這個檔案，因為深怕 CLIENT_ID 被覆蓋掉。

注意到了嗎？你做了一個**擴展性很差**的決定，因為如果未來有更多變數需要依賴於執行環境，你就會需要關注更多檔案，這對懶人工程師而言是相當不友善的。

趁早為專案引入 .env 吧！這些額外心力是完全可以避免的。

### 那麼要如何根據環境引入不同的.env 呢？

如果你使用 CRA 建立專案，可以參考這個[文件](https://create-react-app.dev/docs/adding-custom-environment-variables/#what-other-env-files-can-be-used)。

> 如果不是...那你可能要稍微了解一下 Webpack DefinePlugin，大部分的環境變數套件都是靠它實作的。

簡單說，CRA 專案會在你執行指令時自動判斷要引入哪個對應的.env 檔 (權重由大到小):

- npm run start : .env.development.local, .env.local, .env.development, .env
- npm run build : .env.production.local, .env.local, .env.production, .env
- npm run test : .env.test.local, .env.test, .env(note .env.local is missing)

所以我們可以在專案內新增一個.env.development 檔案:

```
# .env.development
REACT_APP_CLIENT_ID = myLocalClientId
```

一個.env.production 檔案:

```
# .env.production
REACT_APP_CLIENT_ID = myProductionClientId
```

> ### 注意：一定要加上 REACT_APP 前綴 CRA 才會將這個變數打包進 process.env

細心的你可能注意到了，那 uat 環境呢？這樣我們只設定了兩個變數版本呀？

### 自行擴充.env

要自行針對需求來擴充 env 種類，我認為最簡單的方式是使用 [dotenv-cli
](https://www.npmjs.com/package/dotenv-cli) 套件。

> 當然有志者也可以直接透過`react-app-rewired`套件編輯`config-overrides`檔案，自行編寫 webpack 規則。

這邊我們的需求是新增一個 uat 的環境變數:

```
// package.json
"scripts": {
  "start": "dotenv -e .env.development react-app-rewired start",
  "build:uat": "dotenv -e .env.uat react-app-rewired build",
  "build": "dotenv -e .env.production react-app-rewired build",
  "test": "react-app-rewired test",
}
```

```
# .env.uat
REACT_APP_CLIENT_ID = myUatClientId
```

如此一來便可以應對多種環境變數的情況了。<br>
如果你想在開發者模式下模擬線上環境，也可以直接加上:<br>

```
"start:prod": "dotenv -e .env.production react-app-rewired start",
```

不過要注意的點是，此情況下 webpack 預設的 NODE_ENV 還是 development，如果程式內有進行判斷的要特別小心。

有了 `dotenv-cli` 可以讓我們的環境變數擴充更方便，應對複雜的專案需求也可以任意混搭，實在是很方便！

### 接下來...?

如果是手動部署，你現在已經可以透過在三台機器下不同的指令來讓程式引入不同的環境變數了。<br>

但我認為要完全解放雙手，當一個懶人工程師 😎，需要搭配 CI/CD 才算是大功告成，這部分就期待我後續的文章吧！
