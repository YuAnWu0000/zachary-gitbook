# CSS Framework 到底怎麼選？Tailwind css ? styled components ? CSS Module ? (1)

對於我這種選擇障礙來說，看到這麼多選項只覺得頭昏眼花，不然就...

### 與其攤開表格來比較各項優缺，我更喜歡直接用實作場景來切入...

假設我們今天有個 Button component，先來個最傳統的版本 (以 React 為例)：<br>

```
// jsx
function MyButton() {
  return <div className="button">Button</div>
}
// how to use
<MyButton />
```

```
/* css */
.button {
  width: 120px;
  height: 30px;
  fontSize: 14px;
  fontWeight: 500;
  color: black;
}
```

接下來，我們根據 **Design Guideline** 將它設計成三個 props 的組件<br>
分別是: `variant` (決定種類 EX: primary, warning, error), `size` (決定大小), `status` (決定是否可點擊) <br>

### 就組件管理而言, 如何將組件的 props 跟 style 綁定一直是個重大課題

##### 對於這點 CSS module 可以透過一些技巧來綁定對應的 class:

```
// jsx
export function MyButton({ variant, size, status }) {
  return
    <div className={`button button-${variant} button-${size} button-${status}`>
      Button
    </div>
}
// how to use
<MyButton variant="primary" size="big" status="disable"/>
```

```
/* sass */
.button {
  width: 120px;
  height: 30px;
  fontSize: 14px;
  fontWeight: 500;
  color: white;
  &-primary: {
    font-weight: bold;
    background-color: black;
  }
  &-big {
    width: 200px;
    height: 50px;
  }
  &-disable {
    pointer-events: none
  }
}
```

覆蓋順序是相當直觀的後蓋前，也就是如果 `button-primary` 跟 `button-disable` 有重複的 style，會是後宣告的 `button-disable` 贏。

##### 而 styled components (emotion) 則是:

```
// jsx
import styled from '@emotion/styled'
import { css } from '@emotion/react'

const ButtonDefault = (props) => css`
  width: 120px;
  height: 30px;
  fontSize: 14px;
  fontWeight: 500;
  color: white;
  pointer-events: ${props.isDisable ? 'none' : 'auto'};
`
const PrimaryButton = styled.div`
  ${ButtonDefault};
  font-weight: bold;
  background-color: black;
`
const BigButton = styled.div`
  ${ButtonDefault};
  width: 200px;
  height: 50px;
`

// how to use
<PrimaryButton isDisable={true} />
<BigButton isDisable={true} />
```

> ##### _你可能會納悶，為什麼要把 CSS 的事情挪到 JS 裡面來做呢？你看他到頭來還是寫 css syntax 呀？_

### 其實所有的 css in js 解決方案本質上只有一個目的，那就是利用 JS 的特性來更好的管理、抽換 style

以上面的例子來說，假設我們今天需要 `button` 多一種 `disable` 的狀態，那兩種方案的順序會是：<br>

**CSS module： 新增 `.button-disable` 的 class -> 透過 `button-${props.status}` 的方式來綁定 classname -> 傳入 `status` props 即可連動 classname**<br>

**styled components： 在原本 `button` 的 class 內部新增一行三元判斷 -> 傳入 `isDisable` props 即可連動該 `pointer-events` 屬性**

---

你會發現 css in js 不再需要額外透過一些技巧綁定 classname 了，因為他的 css 定義在 js 內，所以可以**直接拿 props 來判斷屬性值就好。**

以開發體驗來說，**styled components** 少了一些思考如何 **binding classname** 的心智煩惱，也多了更多的 js 特性可以作為武器使用 (ex: 三元、Object)，看起來簡直完美！

但眼尖的讀者應該有發現，這邊的寫法跟 CSS module 有些許不同。

```
// how to use
<PrimaryButton isDisable={true} />
<BigButton isDisable={true} />
```

**現在我們的 `button` 不能同時是 `primary` 也是 `big` 了。**

這是因為我們把 `PrimaryButton` 與 `BigButton` 分開定義的緣故。<br>
如果你想要做出 CSS module 那種 `class="button button-primary button-big"` 的效果，這種寫法有可能會需要多個三元表達式，因為 `size="big"` 會同時影響 `width`, `height` 等多個屬性。<br>

例如下面這樣:

```
const ButtonDefault = (props) => css`
  width: ${props.isBig ? '200px' : '120px'};
  height: ${props.isBig ? '50px' : '30px'};
  fontSize: ${props.isBig ? '16px' : '14px'};
  fontWeight: 500;
  color: white;
  pointer-events: ${props.isDisable ? 'none' : 'auto'};
`
```

而且當兩個以上的 props 都會影響一個 property 的時候，還會變得更醜。

既然如此，那我們來試試 Emotion 官方提供的另一種 css prop 的實作方式吧:

```
import { css } from '@emotion/react'

const buttonCSS = css({
  width: '120px',
  height: '30px',
  fontSize: '14px',
  fontWeight: 500,
  color: 'white'
})

const PrimaryButtonCSS = css({
  fontWeight: 'bold',
  backgroundColor: 'black'
}, buttonCSS)

const PrimaryBigButtonCSS = css({
  width: '200px',
  height: '50px',
}, PrimaryButtonCSS)

const BigButtonCSS = css({
  width: '200px',
  height: '50px',
}, buttonCSS)

export function MyButton({ variant, size, status }) {
  if (variant === 'primary') {
    if (size === 'big')
      return
        <div css={PrimaryBigButtonCSS}>
          Button
        </div>
    else
      return
        <div css={PrimaryButtonCSS}>
          Button
        </div>
  }
  else {
    if (size === 'big')
      return
        <div css={BigButtonCSS}>
          Button
        </div>
    else
      return
        <div css={buttonCSS}>
          Button
        </div>
  }
}

// how to use
<MyButton variant="primary" size="big"/>
```

基本上跟 styled components 差不多，要馬使用多個三元表達式寫在一起，要馬 if else 直接硬幹，但都沒辦法很優雅的應對持續增加 props 的情況，我認為這是所有 **css-in-js** 的硬傷：**一旦當我們需要 apply 多個相依於 props 的 class，事情就容易變得很複雜**。

以上，對於 **CSS module** 以及 **styled components** 的介紹就先點到為止，相信讀者已經有點感覺了，我知道忽略了很多面相，但我暫時不想讓討論太發散，還請各位保持耐心。

下一篇同樣會以這個例子來著重講講 **Atomic css**。
