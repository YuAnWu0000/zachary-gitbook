# 自架 gitlab-runner，搞定 gitlab-ci，從此一勞永逸吧

在開始這一切之前，有人還不了解 gitlab CI 運作機制的嗎？我先在這邊附上艦長製的圖，非常淺顯易懂：<br>
<img src="../../images/gitlab-ci/runner.png" width="450" >

簡單說，當你跟 Gitlab 做出互動，例如: 下 tag 或是 push commit，Gitlab 會自動觸發你專案中的 `gitlab-ci.yml`。<br>
接著 Gitlab 會需要找一台 server 來執行`gitlab-ci.yml`裡面的指令(Jobs)，這個執行的 server 就叫做 gitlab-runner。<br>
最後 Gitlab 會將 gitlab-runner 的執行過程跟執行結果顯示於 Pipeline 給你看。

### 1. Install gitlab-runenr

```
# Linux x86-64
sudo curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
```

### References

[GitLab CI 之 Runner 的 Executor 該如何選擇？](https://chengweichen.com/2021/03/gitlab-ci-executor.html)
[如何從頭打造專屬的 GitLab CI/CD](https://pin-yi.me/blog/git-or-cicd/gitlab-cicd/#%e8%87%aa%e6%9e%b6-runner-specific-runners)<br>
[Install GitLab Runner manually on GNU/Linux](https://docs.gitlab.com/runner/install/linux-manually.html)
