# 回答一題 var / let / const 就驚艷你的面試官 ─ 滿分標準

接續上一篇的問題......

### Hoisting

> MDN: JavaScript Hoisting refers to the process whereby the interpreter appears to move the declaration of functions, variables, classes, or imports to the top of their scope, prior to execution of the code.

**Hoisting = move the declaration to the top of their scope.**，就是這麼簡單的一句話，卻引來了許多爭議，原因就是，hoisting 並非一個官方詞彙，它並沒有被寫進 ECMAScript 規格當中。

> _MDN: Hoisting is not a term normatively defined in the ECMAScript specification._

**彷彿在程式的世界見到「一中各表」一樣，在 Javascript 的世界裡，也有著「一詞提升，各自表述」的有趣現象。**

前面的例子中：

```
var a = 123
if (true) {
  a = 456 // Uncaught ReferenceError: Cannot access 'a' before initialization at <anonymous>:3:7
  let a
}
```

我們可以發現 JS interpreter 實際上在執行程式碼之前，有針對 `let` 的宣告做類似「半提升」的行為，為什麼會說只做了一半呢？因為跟 `var` 相比， 我們沒辦法提早取得`undefined`的值，我們能做的就只有從錯誤訊息裡面分辨：「喔~下一行的 let declaration 竟然影響到上一行的賦值了！」

要能更清楚的理解問題的癥結點，我認為必須簡單講解一下變數的生命週期：
