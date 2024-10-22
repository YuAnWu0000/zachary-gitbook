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

- stages: 定義這個 pipeline 有幾個不同階段，這個例子只有一個`deploy` (你在 Gitlab/CI/CD/Pipelines 就只會看到一個圈圈)。
- deploy-job: 定義了一個 job 名稱為 `deploy-job`，內部可指定對應到哪個 stage。
- script: 逐行執行的指令。
- $GITLAB_USER_LOGIN: 此為 **Predefined Variables by Gitlab**，總共有哪些可參考[官方文件](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html#predefined-variables)。
