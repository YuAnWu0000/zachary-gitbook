# 回答一題 var / let / const 就驚艷你的面試官 ─ 滿分標準

接續上一篇的問題...

### Hoisting

> MDN: JavaScript Hoisting refers to the process whereby the interpreter appears to move the declaration of functions, variables, classes, or imports to the top of their scope, prior to execution of the code.

簡單說，**Hoisting = You can access variable before declaration.**<br>
但就是這麼簡單的一句話，卻引來了許多爭議 😥。<br>
因為，hoisting 並非一個官方詞彙，沒有被寫進 ECMAScript 規格當中，它是為了方便我們這些 JS 使用者理解流程而被發明出來的，所以在定義方面一直是各說各話。<br>

> _MDN: Hoisting is not a term normatively defined in the ECMAScript specification._

**彷彿在程式的世界見到「一中各表」一樣，在 Javascript 的世界裡，也有著「一詞提升，各自表述」的有趣現象。**

### 要能有效地切入問題的癥結點，我認為可以把變數的初始化分成三階段來理解：

##### 1. create

- 建立作用域 / 紀錄變數名 / 預留每個變數的記憶體空間

##### 2. initialize

- 為所有宣告的變數第一次指定值 (注意：此動作並非賦值)，並且把沒有在 declaration 給值 (initializer) 的變數默認塞給他 `undefined`，例如 `var a` 預設 `a = undefined`

##### 3. assign

- 執行程式，處理所有的程式碼中賦值行為

> 接著讓我們來看看例子：

```
console.log(a) // undefined
var a = 123
```

`var` 之所以被認為有 hoisting 是因為 JS interpreter 把 **create + initialize** 都提早做完了，**So you can
access "a" before declaration.** (儘管取得的值是 undefined，但起碼不會報錯。)

> 那麼 `let` 呢？

```
console.log(a) // Uncaught ReferenceError: a is not defined
let a = 123
```

單純看這個簡單的例子你會認為`let`沒有 hoisting，但請你看看接下來的這個例子：

```
var a = 123
if (true) {
  a = 456 // Uncaught ReferenceError: Cannot access 'a' before initialization at <anonymous>:3:7
  let a
}
```

**如果真的沒有「提升」，那為什麼程式會在第三行報錯？照理說應該可以直接把全域的 a 賦值為 456 才對。**<br>

因此我們只能推斷 JS interpreter 實際上在執行程式碼之前，有針對 `let` 的宣告做類似「半提升」的行為，為什麼會說只做了一半呢？因為跟 `var` 相比，它只預先做了 **create** 的步驟，也因此我們沒辦法取得初始化後 `undefined` 的值，但從 TDZ 來判斷， 這個 **create** 的動作已經讓整個 block scope 都無法再使用 a 變數了。

> 順帶一提如果是 function declaration 的情況

```
catName("Chloe");
function catName(name) {
  console.log("My cat's name is " + name);
}
/*
上面程式的結果是: "My cat's name is Chloe"
*/
```

則是幫你把 **create + initialize + assign** 都做完了，也因此你可以在 scoped 內的任何地方正常使用`catName()`。

### 所以...`let` 到底有沒有 hoisting？

我認為這取決於你自己對 hoisting 的定義，如果你認為 hoisting 至少就是得 create + initialize，那確實，`let`沒有 hoisting，那如果你的理解比較抽象跟廣義，認為 TDZ 本身就是一個 hoisting 的證明，我也覺得沒問題。

各位啊！真的不是我故意在這打太極不給個明確的答案，你們自己看看 MDN 在 hoisting 頁面前後兩段的文字就知道他們自己的立場也蠻飄忽不定的：

> However, the temporal dead zone can cause other observable changes in its scope, which suggests there's some form of hoisting

> Still, it may be more useful to characterize lexical declarations as non-hoisting, because from a utilitarian perspective, the hoisting of these declarations doesn't bring any meaningful features.

而且最後站邊 non hoisting 的理由竟然是這個現象並不「實用」？用結果論來定義一個為了幫助理解過程而誕生的詞彙 ─ Hoisting，我怎麼想都覺得有點怪怪的。

### 如果我是面試官

各位可以發現，這個問題已經漸漸的從是非題變成了申論題，每個人都有他各自的立場。如果我是面試官，其實並不會太注重你的最終答案是 hoisting or non-hoisting，而是從你分析的過程下去觀察你是如何定義問題、架構邏輯、表達立場的，這些，才是一個工程師不可或缺的軟實力。
