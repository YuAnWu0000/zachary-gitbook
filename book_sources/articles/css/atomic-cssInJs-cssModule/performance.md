# CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? ─ 談效能 (4)

### 淺談效能

談到前端效能無外乎就是以下兩點：

1. bundle size
2. run time

##### 1. bundle size

Atomic css 的特點就是每個屬性值都只會有一個 utility class，因此一個專案就算用到 50 個 `display: flex`，也只是重複使用 `flex` 這個 class 罷了，相比傳統的 css module 把每個需要樣式的節點都量身定義一個 class，這種作法減少了非常多的樣式重複定義，因此有效降低了 css bundle size。
另外前面有提到 css module 由於需要開發者特別關注權重跟覆蓋性的問題，有時候容易留下一些沒有使用到的 class，導致 bundle size 的增加，在維護長期專案的情況下這點也是不容忽視的。

##### 2. run time

相信大家都有一個共識，那就是在 run time 成本上，越貼近原生瀏覽器的作法就越快。
由於 scss 等 css module 解決方案都是在 compile time 做靜態分析將樣式編譯為 pure css，所以這種最貼近原生的作法 run time 自然最快。
`Tailwind css`由於需要使用`tw-merge`來進行字串合併，因此多了一些字串比對跟查找的額外功夫。
`css-in-js`大概是 run time 成本最高的一種解決方案，因為所有的樣式都定義在 js 當中，實際程式在跑的時候需要動態組合樣式並產生對應的 class 然後 binding 到模板上，在面對頻繁抽換 style 的情境有可能會出現效能上的問題。
