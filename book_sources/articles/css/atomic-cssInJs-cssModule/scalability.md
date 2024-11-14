# CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? (3) ─ 談擴展

### 接著讓我們來談談可擴展性 ☕ (scalability)

基於前面的案例，思考下面這幾種擴充情境

1. 增加一個 prop `isHidden`，設置`display: none`。
2. 增加一個 prop `color`，直接讓外部決定顏色為何。
3. 讓外部可以傳客製化的`className`進來。

##### 1. 增加一個 prop `isHidden`，設置`display: none`

**CSS module:** 新增一個 class，然後透過三元來判斷是否加入<br>

```
/* scss */
.hidden {
  display: none;
}
// jsx
<div
  className={`button button-${variant} button-${size} button-${status}
    ${isHidden ? 'hidden' : ''}`} // 新增一個三元
>
  Button
</div>
```

**css-in-js:** 單個 prop 影響單一屬性的情況下，新增一行三元表達式輕鬆解決。<br>

```
const buttonStyles = css({
    ...
    display: `${isHidden ? 'none' : 'block'}`
  })
```

**Atomic css:** 同樣是新增一行三元表達式輕鬆解決。<br>

```
let totalStyles = twMerge(
  ...
  isHidden ? 'hidden' : '',
)
```

##### 2. 增加一個 prop `color`，直接讓外部決定顏色為何

**CSS module:** 因為 CSS module 的設計是以 class 為最小單位，不好直接應對屬性值。<br>
因此除了`style={{ color }}`幾乎沒有太優雅的方式，但行內樣式又有著難以被覆蓋的問題 😓。<br>

**css-in-js:** 直接將 prop 塞進 css object，結束。<br>

```
const buttonStyles = css({
    ...
    color
  })
```

**Atomic css:** 大概又是新增一行就 OK 了吧...<br>

```
let totalStyles = twMerge(
  ...
  `text-${color}-500`,
)
```

這時候你發現顏色卻沒有變 😥，打開 console 一查發現 className 有指定對，但是沒有對應的 class。<br>

恭喜你又踩到一個坑，原因是 **Tailwind 會在 compile time 透過字串掃描來決定整個專案會用到哪一些 utility class**，因此這種字串拼接的方式並不會被 scan 到，該 class 也不會被打包進來，顏色當然也就不會變了。<br>

兩種方式解決：要馬就直接把`color-yellow-500`當成 prop 傳入，要馬就直接傳入自定義的 className，不要讓外部只傳屬性進來，畢竟 Atomic css 在設計思想上的最小單位也是 class。

##### 3. 讓外部可以傳客製化的`className`進來

**CSS module:** 新增一個 className prop 放入子組件的 className 即可。<br>

```
/* scss */
/* 定義在最後面 */
.button-special {
  color: blue;
  background-color: red;
  display: block;
}
// 子組件內
<div
  className={`button button-${variant} button-${size} button-${status}
    ${isHidden ? 'hidden' : ''} ${className}`}
  style={{ color }}
>
  Button
</div>
// 外部使用
<MyButton
  variant="primary"
  size="big"
  status="disable"
  isHidden={true}
  color="yellow"
  className="button-special" // 客製化class
/>
```

來，猜猜看，出來的 button 長什麼樣子？<br>
.<br>
.<br>
.<br>

![button images](../../../images/atomic-cssInJs-cssModule/buttonCssModule.png)

嗯？好像有點怪怪的？原因如下：<br>
首先，我們把 `.button-special` 定義在最底下，因此在同階層的情況下它的優先級比 `.hidden` 高，這也是為什麼儘管設定了 `isHidden={true}` 但依然不管用的原因。<br>

另外前面提到的 `color prop`，因為組件內部是直接套用 `style={{ color }}`，因此它的優先級最高，導致我們無法將字體顏色變更為 `.button-special` 內部定義的藍色。

**由這個例子可以看出，如果樣式出現非預期的結果，CSS module 需要同時關注模板 & 樣式 (這通常是兩個檔案)，其中尤其是 CSS，你必須確認順序性跟權重的問題，我認為這就是它 DX 相比於其他兩者不太友善的地方。**

**css-in-js:** 在子組件內新增一行輕鬆解決 👊。(正常來說應該是外部把 css prop 傳入啦，在這邊為了方便比較就先統一命名成 className)<br>

```
// 子組件內
const buttonStyles = css({
    ...variantButtonCSS[variant],
    ...sizeButtonCSS[size],
    ...(isDisable ? disableCSS : null),
    display: `${isHidden ? 'none' : 'block'}`,
    color,
    ...className // 新增這行
  })
```

```
// 外部使用
<MyButton
  variant="primary"
  size="big"
  isDisable={false}
  isHidden={true}
  color: "yellow"
  className={{ //客製化class
    color: 'blue',
    backgroundColor: 'red',
    display: 'block'
  }}
/>
```

結果：<br>
![button images](../../../images/atomic-cssInJs-cssModule/buttonCssInJs.png)<br>
非常好！完全是我們預期的行為！**而要預測此行為只需要關注模板內 object 的組合順序就可以了！**

**Atomic css:** 同樣是新增一行輕鬆解決 👌。<br>

```
// 子組件內
let totalStyles = twMerge(
  defaultStyles,
  variantStyles[variant] ? variantStyles[variant] : '',
  sizeStyles[size] ? sizeStyles[size] : '',
  isDisable ? disableStyles : '',
  isHidden ? 'hidden' : '',
  textColor,
  className // 新增這行
)
```

```
// 外部使用
<MyButton
  variant="primary"
  size="big"
  isDisable={false}
  isHidden={true}
  textColor="text-yellow-500"
  className="text-blue-500 bg-red-500 block" // 客製化class
/>
```

結果：<br>
![button images](../../../images/atomic-cssInJs-cssModule/buttonAtomicCss.png)<br>
同樣是預期內的結果，我們也只需要關注模板本身就好了。

---

由上面的例子可以清楚看出三種 CSS framework 在面對業務擴展時的 scalability，**我認為在三者當中就屬 css-in-js 的功能最強大 💪**，因為它利用 JS 的語法特性可以更優雅的應對各種不同的場景，我想這也是這麼多 UI library 採用 css-in-js 作為解決方案的原因，畢竟在面對眾多用戶的前提下，**抽象化 (abstraction)、彈性 (flexibility)、可擴展性 (scalability)，就會是首要考量。**<br>

當然其他兩者也不算差，Tailwind 撇除偶爾會遇到奇怪的坑，搭配上 `twMerge` 也有很不錯的彈性，至於 CSS module 則是需要額外關注 css file 中 class 定義的順序性跟權重等等，如果有些 class 長期被覆蓋掉可能也不好發現，面對日漸複雜的業務有可能會留下比較多的 legacy code。

下一篇就讓我們來談談效能吧！敬請期待！

**下集:**

- [CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? (4) ─ 談效能](https://yuanwu0000.github.io/zachary-gitbook/articles/css/atomic-cssInJs-cssModule/performance.html)
