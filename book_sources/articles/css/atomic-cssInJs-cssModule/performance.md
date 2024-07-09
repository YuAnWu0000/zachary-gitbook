# CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? (4) ─ 談效能

### 淺談效能

談到前端效能無外乎就是以下兩點：

1. Bundle size
2. Run time

##### 1. Bundle size

Atomic css 的特點就是每個屬性值都只會有一個 utility class，因此一個專案就算用到 50 個 `display: flex`，也只是重複使用 `flex` 這個 class，相比傳統的 css module 把每個節點都量身定義一個 class，**Atomic 的作法減少了非常多的樣式重複定義，因此有效降低了 css bundle size。**<br>
另外，css module 在開發時還需要特別關注權重跟覆蓋性的問題，在專案功能越做越複雜的情況下，難免會出現一些 zombie styles，這些沒有被使用到的 class 會導致 bundle size 的增加，需要開發者額外小心注意。<br>
至於 css-in-js 的解決方案如 Emotion ，bundle size 只有 7.9 KB，不算什麼太大問題。

##### 2. Run time

相信大家都有一個共識，那就是在 run time 成本上，**越貼近原生瀏覽器的作法就越快。**<br>
由於 scss 等 css module 解決方案都是在 compile time 做靜態分析將樣式編譯為 pure css，所以 run time 時候其實跟原生沒有太大區別。<br>
Tailwind css 有時需要使用 `tw-merge` 來進行字串合併，因此多了一些字串比對跟查找的額外 run time 成本，但除此之外並沒有其他太大的負擔。<br>
css-in-js 大概是 run time 成本最高的一種解決方案，因為所有的樣式都定義在 js 當中，實際程式在跑的時候需要動態組合樣式並產生對應的 class 最後才 binding 到模板上，因此，**面對頻繁抽換 style 的情境有可能會出現效能上的問題。**編譯出來的結果則是跟 CSS module 類似： `classname-{hash}`，但相對來說 zombie styles 要少得多，畢竟不需要太過關注權重的問題。

### 結論

**以學習曲線來說：** CSS module > Emotion > Tailwind<br>
**以開發者體驗來說：** Tailwind > CSS module > Emotion<br>
**以偵錯體驗來說：** CSS module > Emotion > Tailwind<br>
**以擴展性來說：** Emotion > Tailwind > CSS module<br>
**以效能來說：** Tailwind >= CSS module > Emotion<br>

上述的比較純屬個人看法，投資框架有賺有賠，使用前應詳閱公開說明書。<br>
我認為重點還是要知道自己為什麼選擇當前的 CSS framework，**並且知道手上的工具有什麼短版：**<br>

比如你喜歡 css-in-js 提早抽象化的寫法，但又擔心有 performance bottleneck，那麼也許你可以採用 Zero-Runtime 的解決方案如：

- [linaria](https://linaria.dev/)
- [Compiled](https://compiledcssinjs.com/)

又或者你使用 CSS module、Tailwind 但卻想要更大的可擴展性，那可以試著用 [classnames](https://www.npmjs.com/package/classnames) 來幫助你達成！

> ### 工程師善用手上的工具，而不是辯護手上的工具

在前端領域每天都有新技術出現，身為工程師應該要擁抱改變，而不是畏懼改變，畢竟世界需要我們走在前面:)

到這邊，CSS Framework 系列文結束啦，我用了四篇文章談了實作案例、開發者體驗、可擴展性及效能的差異，希望有讓讀者更清楚要選擇哪一個框架來應對專案需求。

很感謝看到這的你們，如果對這篇文章有其他想法或意見，可以直接開 Github issue 或私訊我的 LinkedIn，相信我們很快就能開啟愉快的技術討論！

最後就祝大家<br>
Happy ~~debating~~ coding！

### References

https://dev.to/srmagura/why-were-breaking-up-wiht-css-in-js-4g9b
