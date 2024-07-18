# 回答一題 var / let / const 就驚艷你的面試官

### 「請解釋 var / let / const 的差別？」

一道經典的前端面試題，告訴我，你的回答是不是只有下列幾種：<br>
「`var` 是 function scope； `let` / `const` 是 block scope。」<br>
「`var` 可以被重複宣告； `let` / `const` 不行。」<br>
「`const` 宣告以後就不能重新賦值。」<br>

最近在準備面試的過程中，我發現這個題目看似簡單，但其實包含著超級多 JS 的基本概念，對面試官來說，Follow up 可以延伸發散到各個不同的議題如：hoisting, variable life cycle, GC, scope and context 等，絕對不是三言兩語就可以打發掉的各類議題。

> 深入展開以前，先讓我們複習一下上面提到的基本特性吧！

### var

- 在全域宣告的變數 `var`, 也會同時作為全域物件(前端為 `window`, 後端為 `global`)的屬性

```
var a = 0
console.log(window.a) // 0
```

- `var` 可重複宣告, 以後宣告的為主。(指的是在同一個作用域底下)

```
var a = 0
var a = 1
console.log(window.a) // 1
```

- 不同作用域的情況, 在 function 裡面宣告的就只存在於該 function scope

```
var a = 0
function test() {
  var a = 1
  console.log(a)
}
test() // 1
console.log(a) // 0
```

### let

- 作用於**Block Scope**

```
if (true) {
  let a = 0
}
console.log(a) // Uncaught ReferenceError: a is not defined
```

- 在**Block Scope**底下有宣告動作，就不管全域如何(**這個錯誤訊息怪怪的，下面會詳細解釋。**)

```
var a = 123
if (true) {
  a = 456
  // Uncaught ReferenceError: Cannot access 'a' before initialization
  let a
}
```

**上面這種在宣告前賦值失敗的情況也稱作暫時性死區(Temporal Dead Zone)**

- 不可重複宣告

```
let a = 123
let a = 456 // Uncaught SyntaxError: Identifier 'a' has already been declared
```

### const

**let 的特性都有, 新增以下:**

- 宣告後就不可更改

```
const a = 123
a = 456 // Uncaught TypeError: Assignment to constant variable.
```

- 也不可單獨宣告不給值

```
const a // Uncaught SyntaxError: Missing initializer in const declaration
```

- 同個作用域底下，不可以重複宣告 const

```
var a = 123
const a = 456 // Uncaught SyntaxError: Identifier 'a' has already been declared
```

- 不同作用域底下則可以重複宣告

```
var a = 0;
function test() {
  const a = 1
  console.log(a)
}
test() // 1
console.log(a) // 0
```

- 指向`Object`或是`Array`的時候更改其屬性不會報錯(store by reference 所以只確保指標沒有變)

```
const foo = { a: 0 }
foo.b = 1 // ok
foo = {} // Uncaught TypeError: Assignment to constant variable.
```

```
const foo = [0]
foo.push(1) // ok
foo = [] // Uncaught TypeError: Assignment to constant variable.
```
