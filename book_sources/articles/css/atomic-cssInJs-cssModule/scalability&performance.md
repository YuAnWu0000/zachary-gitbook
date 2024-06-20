# CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? (3)

### 接著讓我們來談談可擴展性 (scalability)

基於前面的案例，思考下面這幾種擴充情境

1. 增加一個 props `isHidden`，設定`display: none`。
2. 增加一個 props `color`，直接讓外部決定顏色為何。
3. 讓外部可以傳客製化的`className`進來。

##### 1. 增加一個 props `isHidden`，設定`display: none`

CSS module: 新增一個 class，然後透過三元來判斷是否加入<br>

```
// scss
.hidden {
  display: none;
}
// jsx
<div className={`button button-${variant} button-${size} button-${status} $      {isHidden ? 'hidden' : ''}`}>Button</div>
```

css-in-js: 單個 props 影響單一屬性的情況下，新增一行三元表達式輕鬆解決。<br>

```
const buttonStyles = css({
    ...
    display: `${isHidden ? 'none' : 'block'}`
  })
```

Atomic css: 同樣是新增一行三元表達式輕鬆解決。<br>

```
let totalStyles = twMerge(
  ...
  isHidden ? 'hidden' : '',
)
```

##### 2. 增加一個 props `color`，直接讓外部決定顏色為何

CSS module: 除了`style={{ color }}`幾乎沒有太優雅的方式，但行內樣式有難以被覆蓋的問題。<br>

css-in-js: 將 props 塞進 css object，結束。<br>

```
const buttonStyles = css({
    ...
    color
  })
```

Atomic css: 又踩到一個坑，新增一行後發現顏色沒有變？？<br>

```
let totalStyles = twMerge(
  ...
  `text-${color}-500`,
)
```

原因是 Tailwind 會在 compile time 透過字串比對掃描整個專案用到哪一些 utility class 然後打包進來，因此這種字串拼接的方式並不會被 scan 到，於是跑起來的時候沒有對應的 class，顏色當然也就不會變了。<br>
兩種方式解決：要馬就直接把`color-yellow-500`當成 prop 傳入，要馬就直接傳入自定義的 className，像下面這個例子這樣。

### 效能比較

### 結論
