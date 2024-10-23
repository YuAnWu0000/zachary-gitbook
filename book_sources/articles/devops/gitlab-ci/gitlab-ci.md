# 解放工程師的雙手，你需要好的 CI/CD ─ 搞定 gitlab-ci + 環境變數

上一篇搞定 gitlab-runner 的各種設定以後，我們馬上來建立一個簡單的 gitlab-ci 測試看看吧~<br>
喔對了`.gitlab-ci.yml`，檔名前面記得加`.`，不要跟我一樣耍笨 😓。

```
# .gitlab-ci.yml
stages:
  - deploy
deploy-job:
  stage: deploy
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
    - pwd
    - whoami
    - echo "This job deploys from the $CI_COMMIT_BRANCH branch."
  only:
    - master
  tags:
    - test-runner
```

- _**stages**_: 定義這個 pipeline 有幾個不同階段，這個例子只有一個階段 `deploy` (意味著你在 Gitlab/CI/CD/Pipelines 就只會看到一個圈圈)。
- _**deploy-job**_: 定義了一個 job 名為 `deploy-job`，內部可指定它對應到哪個 stage。
- _**script**_: 逐行執行的指令。
- _**$GITLAB_USER_LOGIN, $CI_COMMIT_BRANCH**_: 此為 **Predefined Variables**，總共有哪些可參考[官方文件](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html#predefined-variables)。
- _**pwd**_: 個人覺得顯示當前路徑 Debug 還蠻方便的，如果 pipeline 有什麼異常，可以直接進到機器的該目錄底下檢查，通常是`/home/gitlab-runner/builds/xxxxxxxx/0/your-project`。
- _**only**_: 限制這個 job 在某些條件下進行，**注意條件是 OR**，例如：<br>

```
only:
  - master
  - tags
```

**這個 job 只會在 master 分支或是下 tag 的時候觸發，如果需要 AND 請轉用 `rules:`。**

- _**tags**_: 此 tag 非彼 tag，這是**指定要讓哪個 runner 跑的 tag**，還記得嗎？我們上一篇文章在建立 runner 的時候有輸入 tag，這時候就派上用場了。**注意跟 `only:` 不同，這邊的邏輯是 AND**。例如：<br>

```
tags:
  - test-runner
  - production-runner
```

**代表這個 job 只能在同時是 `test-runner` 又是 `production-runner` 的 runner 上面跑** (每個 runner 可以設定多個 tag)，如果一個 job 找不到符合條件的 runner，你會在 pipeline 看到它**永遠是 pending 的狀態**。

> ### 把你的 commit push 到 master 吧！不久後你就可以看到 pipeline 的成功訊息了。

接著讓我們把需要的功能逐個加入這個 CI 裡面吧~

### 搭配環境變數

如果是 React 專案的話，想必專案內一定會有多個 env 檔案，我們假設專案內有`.env.development`, `.env.uat`, `.env.production`，分別對應到三種不同的執行環境。

> 不知道怎麼擴充使用 env 的，可以參考我之前寫的[這篇文章](https://yuanwu0000.github.io/zachary-gitbook/articles/react/env.html)

這時候，我們要做的就是讓 CI 可以依據不同的 Branch，執行不同的 build 指令，例如在 uat 環境是 `npm run build:uat`，在 production 環境則是 `npm run build`。

一般來說如果你使用 docker executor，可以這樣做：

```
# .gitlab-ci.yml
stages:
  - build
build-for-uat:
  stage: deploy
  image: node:20
  script:
    - npm run build:uat
  only:
    - uat
  tags:
    - uat-runner
build-for-production:
  stage: deploy
  image: node:20
  script:
    - npm run build
  only:
    - master
  tags:
    - production-runner
```

需要注意的是要記得開兩台 runner，一台對應到 uat 環境，而另一台對應到 production 環境。
