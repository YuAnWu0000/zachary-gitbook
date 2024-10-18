# 自架 gitlab-runner，搞定 gitlab-ci，從此一勞永逸吧

在開始這一切之前，有人還不了解 gitlab CI 運作機制的嗎？我先在這邊附上艦長製的圖，非常淺顯易懂：<br>
<img src="../../images/gitlab-ci/runner.png" width="450" >

簡單說，

### 1. Install gitlab-runenr

```
# Linux x86-64
sudo curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
```

### References

[GitLab CI 之 Runner 的 Executor 該如何選擇？](https://chengweichen.com/2021/03/gitlab-ci-executor.html)
[如何從頭打造專屬的 GitLab CI/CD](https://pin-yi.me/blog/git-or-cicd/gitlab-cicd/#%e8%87%aa%e6%9e%b6-runner-specific-runners)<br>
[Install GitLab Runner manually on GNU/Linux](https://docs.gitlab.com/runner/install/linux-manually.html)
