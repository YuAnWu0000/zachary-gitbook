# 回答一題 var / let / const 就驚艷你的面試官 ─ 滿分標準

接續上一篇的問題......

### Hoisting

> MDN: JavaScript Hoisting refers to the process whereby the interpreter appears to move the declaration of functions, variables, classes, or imports to the top of their scope, prior to execution of the code.

簡單說，**Hoisting = You can access variable before declaration.**<br>
但就是這麼簡單的一句話，卻引來了許多爭議。<br>
其主因就是，hoisting 並非一個官方詞彙，它並沒有被寫進 ECMAScript 規格當中，它是為了方便我們這些 JS 使用者理解流程而被發明出來的。<br>

> _MDN: Hoisting is not a term normatively defined in the ECMAScript specification._

**彷彿在程式的世界見到「一中各表」一樣，在 Javascript 的世界裡，也有著「一詞提升，各自表述」的有趣現象。**

### 要能有效的切入問題的癥結點，我認為可以把變數的初始化分成三階段來理解：

##### 1. create

- 建立作用域 / 紀錄變數名 / 預留每個變數的記憶體空間

##### 2. initialize

- 為所有宣告的變數第一次指定值 (注意：此動作並非賦值)，並且為沒有在 declaration 給值 (initializer) 的變數默認塞給他 `undefined`

##### 3. assign

- 執行程式，處理所有的程式碼中賦值行為
  前面的例子中：

> 接著讓我們來看看例子：

```
console.log(a) // undefined
var a = 123
```

`var` 有 hoisting 是因為 JS interpreter 把 **create + initialize** 都提早做完了，**You can
access "a" before declaration.** (儘管取得的值是 undefined，但起碼不會報錯。)

> 那麼 `let` 呢？

```
var a = 123
if (true) {
  a = 456 // Uncaught ReferenceError: Cannot access 'a' before initialization at <anonymous>:3:7
  let a
}
```

我們可以發現 JS interpreter 實際上在執行程式碼之前，有針對 `let` 的宣告做類似「半提升」的行為，為什麼會說只做了一半呢？因為跟 `var` 相比，我們沒辦法提早取得 `undefined` 的值，但從 TDZ 的情況來判斷，它確實是預先做了 **create** 的動作所以才讓整個 block scope 都無法再使用 a 變數。

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

則是提早把 **create + initialize + assign** 都幫你做完了。

### 所以...`let` 到底有沒有 hoisting？

我認為這取決於你對 hoisting 的定義，如果你認為 hoisting 至少就是得 create + initialize，那確實，`let`沒有 hoisting，那如果你的理解比較抽象跟廣義，認為 TDZ 就是一個 hoisting 的證明，我也覺得沒問題。

各位啊！真的不是我故意在這打太極不給個明確的答案，你們自己看看 MDN 在 hoisting 頁面前後兩段的文字就知道他們自己的立場也蠻飄忽不定的：

> However, the temporal dead zone can cause other observable changes in its scope, which suggests there's some form of hoisting

> Still, it may be more useful to characterize lexical declarations as non-hoisting, because from a utilitarian perspective, the hoisting of these declarations doesn't bring any meaningful features.

而且最後站邊 non hoisting 的理由竟然是這個現象並不「實用」？用結果論來定義一個為了幫助理解過程而誕生的詞彙 ─ Hoisting，我怎麼想都覺得有點怪怪的。

### 如果我是面試官