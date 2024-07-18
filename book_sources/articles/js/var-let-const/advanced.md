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
console.log(a)  // undefined
var a = 123
```

`var` 有 hoisting 是因為 JS interpreter 把 create + initialize 都提早做完了，

```
var a = 123
if (true) {
  a = 456 // Uncaught ReferenceError: Cannot access 'a' before initialization at <anonymous>:3:7
  let a
}
```

我們可以發現 JS interpreter 實際上在執行程式碼之前，有針對 `let` 的宣告做類似「半提升」的行為，為什麼會說只做了一半呢？因為跟 `var` 相比， 我們沒辦法提早取得`undefined`的值，我們能做的就只有從錯誤訊息裡面分辨：「喔~下一行的 let declaration 竟然影響到上一行的賦值了！」
