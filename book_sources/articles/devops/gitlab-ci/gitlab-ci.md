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
  only: master
  tags: test-runner
```
