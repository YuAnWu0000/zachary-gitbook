# CSS Framework 到底怎麼選？Tailwind CSS ? styled components ? CSS Module ? ─ 談體驗 (2)

前一篇講到，假設我們有一個接收三個 props 的 button component，以 CSS module 的方式會這樣做:

```
// jsx
export function MyButton({ variant, size, status }) {
  return (
    <div className={`button button-${variant} button-${size} button-${status}`}>
      Button
    </div>
  )
}
// how to use
<MyButton variant="primary" size="big" status="disable"/>
```

```
/* scss */
.button {
  ...
  &-primary: {
    ...
  }
  &-big {
    ...
  }
  &-disable {
    ...
  }
}
```

### 那麼 Atomic css 又是如何將組件的 props 跟 style 綁定的呢？ (以 Tailwind 為例)

```
import { twMerge } from 'tailwind-merge'
const MyButton = ({ variant, size, isDisable }) => {
  const defaultStyles =
    'w-[120px] h-[30px] text-base font-medium text-white bg-black border border-solid'
  const variantStyles = {
    primary: 'bg-green-500 font-medium border-white rounded-lg',
    warning: 'bg-orange-500 font-medium border-white rounded-lg',
    error: 'bg-red-500 font-medium border-white rounded-lg'
  }
  const sizeStyles = {
    big: 'w-[200px] h-[50px] text-lg',
    small: 'w-[100px] h-[20px] text-sm'
  }
  const disableStyles = 'bg-gray-500 pointer-events-none'
  let totalStyles = twMerge(
    defaultStyles,
    variantStyles[variant] ? variantStyles[variant] : '',
    sizeStyles[size] ? sizeStyles[size] : '',
    isDisable ? disableStyles : ''
  )
  return <div className={totalStyles}>Button</div>
}
export default MyButton
// how to use
<MyButton variant="primary" size="big" isDisable={false} />
```

答案是使用 `tailwind-merge` 套件來做合併。<br>

對 Tailwind 來說一切都是字串，造成我們在合併的時候**沒辦法像 css-in-js 利用 Object 的屬性後蓋前，也沒辦法利用原生 CSS class 的後蓋前**，在這種限制之下，只能使用外部套件來幫助我們合併 style。

你可能會覺得有點疑惑，把字串 concat 起來不行嗎？比如：

`` let totalStyles = `${defaultStyles} ${variantStyles[variant]} ${sizeStyles[size]} ${isDisable ? disableStyles : ''}`; ``

這可能是多數人使用 Atomic css 第一個會遇到的坑，那就是**會出現非預期的 css 覆蓋行為**，具體來說，**是你認為應該覆蓋的樣式它卻沒有覆蓋成功。**

還記得在 CSS module 裡面你是怎麼宣告 class 的嗎？

```
.button {
  &-primary: {
    ...
  }
  &-big {
    ...
  }
  &-disable {
    ...
  }
}
```

順序是你自己寫的，很明顯`.button-disable`裡面的樣式就是可以覆蓋`.button-primary`，因為在同階層的情況下，class 定義在後面的贏。<br>
(注意： 就算你外面寫的是`className="button-disable button-primary"`也一樣是`button-disable`會贏，**重點是 class 宣告的順序而不是模板上的順序。)**<br>

然而在 Tailwind 裡面眾多的 utility class 是額外引入的，你並不曉得`w-full w-12`哪個 class 被 Tailwind 定義在後面，因此我們才需要`tailwind-merge`套件來幫我們確保這件事，有了這個套件，我們可以將想要合併的 class string 塞進 `twMerge()` 的 `arguments` 當中，讓套件幫我們處理重複跟順序性的問題。

以上，就是簡單的 Tailwind CSS 實作範例了。

---

不知道各位讀者看完這些實作案例以後有什麼感覺？<br>

> 我個人的看法是，製作組件最常遇到的**將 props 與 style 綁定**的這個課題上，三種 CSS framework 都提供了很成熟的解法。<br>

_**CSS module:** 預先定義好各個 class，利用 props 作為變數 mapping 對應的 classname。_<br>
_**css-in-js (emotion):** 把每個 class 定義為 object，利用 object、spread operator 等 JS 特性把 style 組合起來。_<br>
_**Atomic css (Tailwind):** `button` 的每種狀態都是一個 string (class list)，利用`tailwind-merge`套件來做字串組合。_<br>

我認為在這個議題上，說哪個 framework 不好用都有點牽強，但或許我們可以在這些程式碼基礎上，針對其他議題下去做分析。

### 關於學習曲線

幾乎所有前端的起手式都是從 html, css, javascript 開始學習，想當然**貼近原生寫法的 CSS module 一定是學習曲線最和緩的選擇。**<br>

**css-in-js 的方案需要對 javascript 有一定程度的熟悉才能運用得當**，以前面的例子來說，你如果不知道 spread operator 可以搭配 ternary operator 使用，那你可能會寫出一堆 if else 的程式碼來做 props 的判斷，因此也不算太適合新手。<br>

而 Tailwind，說實話它的學習曲線就是一齣悲劇，首先你要對 CSS 語法熟悉 (不然你根本不知道 syntax 要怎麼查)，再來你需要適應他一堆 boilerplate，最後如果是新手第一次遇到樣式合併的坑一定會很矇。社群中有不少人對 Atomic 很排斥，我不會覺得這是一種不願意學習新東西的藉口，因為**它的學習曲線真的相對其他兩者陡峭許多。**<br>

### 論開發者體驗 (Developer Experience)、語法簡潔程度

開發者體驗(Developer Experience)又簡稱 DX，其實跟語法簡潔有很大的正相關，一個普遍認為 DX 最友善的語言 ─ Python，就是得利於它的語法簡潔性，所以我這邊想放在一起討論。

**Tailwind 在實作上幾乎擁有最簡短的程式碼**，其主因是它把樣式從`background-color: black;`轉設計為`bg-black`這種 utility class，比 CSS module, css-in-js 要來的簡潔許多，犧牲了學習曲線換來的是極佳的開發者體驗。

再來，跟 CSS module 相比，你不再需要兩個 file 切來切去了，對比 css-in-js，你甚至連 JS 的地方都不用看，從頭到尾只需要關注模板的部分。並且新增了 class 以後你再也不需要確認那個 class 內部的樣式有什麼，因為它的 class name 本身就是 self explanation。

**Atomic css 比較會為人詬病的是定位 Bug 的能力**，因為當你遇到樣式出現問題，失去了解釋性的 classname 會讓你不好確定這是哪個 component，然後當你打開 console 看到一坨 utility class，想到接下來只能逐一確認，躁起來是難免的 XD。

相比之下，我認為 CSS module 有著最好的偵錯體驗，因為命名得當的 class name 可以讓你單看模板就快速定位問題組件，並且找出是哪個 class 出現問題，這也是許多人推崇 CSS module 的原因。

但是，一個專案在開發跟找 bug 的時間是極度不平衡的，你可能會花 80% 的時間專注在從 0 到 1 的開發，只有 20% 時間在找 bug，而其中關於樣式相關 bug 的比例可能更低。

因此如果要我選一個 DX 最友善的框架，我還是會投給 Tailwind，他會讓你在 0 到 1 的過程相當舒服 (這句話好像怪怪的)。

然後在 DX 這塊幾乎沒有 css-in-js 的事，因為...它就真的有點囉唆。
