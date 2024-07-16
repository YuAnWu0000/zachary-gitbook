# 回答一題 var / let / const 就驚艷你的面試官

### 「請解釋 var / let / const 的差別？」

一道經典的前端面試題，告訴我，你的回答是不是只有下列幾種：<br>
「`var` 是 function scope； `let` / `const` 是 block scope。」
「`var` 可以被重複宣告； `let` / `const` 不行。」
「`const` 宣告以後就不能重新賦值。」

最近在準備面試的過程中，我發現這個題目看似簡單，但其實包含著超級多 JS 的基本概念，對面試官來說，Follow up 可以延伸發散到各個不同的議題如：hoisting, variable life cycle, GC, scope and context 等，絕對不是三言兩語就可以打發掉的各類議題。

### 先讓我們複習一下上面提到的基本特性吧！

##### 在全域宣告的變數 var, 也會同時作為全域物件(前端為 window, 後端為 global)的屬性

```
var a = 0
console.log(window.a) // 0
```
