# 回答一題 var / let / const 就驚艷你的面試官 ─ 語法特性篇

### 「請解釋 var / let / const 的差別？」

一道經典的前端面試題，告訴我，你的回答是不是只有下列幾種：<br>
「`var` 是 function scope； `let` / `const` 是 block scope。」<br>
「`var` 可以被重複宣告； `let` / `const` 不行。」<br>
「`const` 宣告以後就不能重新賦值。」<br>

最近在準備面試的過程中，我發現這個題目看似簡單，但其實包含著超級多 JS 的基本概念，對面試官來說，follow up 可以延伸到各個不同的議題如：hoisting, variable life cycle, GC, scope and context 等比較有鑑別度的題目，如果只回答得出上面的答案實在有點可惜。

> 深入展開以前，先讓我們複習一下變數基本特性吧！畢竟要想考到滿分，得先及格。 (這什麼廢話 😝)

### var

- 在全域宣告的變數 `var`，也會同時作為全域物件(前端為 `window`，後端為 `global`)的屬性

```
var a = 0;
console.log(window.a); // 0
```

- `var` **可重複宣告**，以後宣告的為主。(指的是在同一個作用域底下)

```
var a = 0;
var a = 1;
console.log(window.a); // 1
```

- 不同作用域的情況，在 function 裡面宣告的就**只存在於該 function scope。**

```
var a = 0;
function test() {
  var a = 1;
  console.log(a);
}
test(); // 1
console.log(a); // 0
```

- 順帶一提，下面這個範例你認為輸出為何?

```
function test() {
  a = 1;
}
test();
console.log(a);
```

答案是: 1，原因是在非嚴格模式下，`a = 1` 若找不到賦值的變數 `a`，**則會直接被"隱含式宣告"成全域變數。**<br>
注意: 如果採用隱含式宣告是**可以被 `delete` 刪除的**，因為它視為幫 windows 新增屬性，相對的其他宣告方式皆無法被 `delete`。

- 但如果多加一行 `var a`...

```
function test() {
  a = 1;
  var a;
}
test();
console.log(a);
```

答案就會變成: `Uncaught ReferenceError: a is not defined`，**因為這次在 function scope 底下找到 var 變數去賦值了(var 有 hoisting)，所以 `a` 不會變成全域變數。**

### let

- 作用於**Block Scope**

```
if (true) {
  let a = 0;
}
console.log(a); // Uncaught ReferenceError: a is not defined
```

- 在**Block Scope**底下有宣告動作，則同名變數不得再宣告前賦值 **(這個錯誤訊息怪怪的，下面會詳細解釋。)**

```
var a = 123;
if (true) {
  a = 456; // Uncaught ReferenceError: Cannot access 'a' before initialization
  let a;
}
```

**上面這種在宣告前賦值失敗的情況也稱作暫時性死區 TDZ(Temporal Dead Zone)**

- 不可重複宣告

```
let a = 123;
let a = 456; // Uncaught SyntaxError: Identifier 'a' has already been declared
```

### const

**let 的特性都有, 新增以下:**

- 宣告後就不可更改

```
const a = 123;
a = 456; // Uncaught TypeError: Assignment to constant variable.
```

- 也不可單獨宣告不給值<br>
  > [ECMA 262 15th](https://262.ecma-international.org/15.0/index.html#prod-BindingList): It is a Syntax Error if _Initializer_ is not present and IsConstantDeclaration of the _LexicalDeclaration_ containing this _LexicalBinding_ is true.

```
const a; // Uncaught SyntaxError: Missing initializer in const declaration
```

- 同個作用域底下，不可以重複宣告 const

```
var a = 123;
const a = 456; // Uncaught SyntaxError: Identifier 'a' has already been declared
```

- 不同作用域底下則可以重複宣告

```
var a = 0;
function test() {
  const a = 1;
  console.log(a);
}
test(); // 1
console.log(a); // 0
```

- 指向`Object`或是`Array`的時候更改其屬性不會報錯(store by reference 所以只確保指標沒有變)

```
const foo = { a: 0 };
foo.b = 1; // ok
foo = {}; // Uncaught TypeError: Assignment to constant variable.
```

```
const foo = [0];
foo.push(1); // ok
foo = []; // Uncaught TypeError: Assignment to constant variable.
```

### 接著開始上點強度

- 為何同樣的例子 `var` 不會噴錯？

```
console.log(a); // undefined
var a = 123;
```

```
console.log(a); // Uncaught ReferenceError: a is not defined
let a = 123;
```

> MDN: In ECMAScript 2015, let bindings are not subject to Variable Hoisting, which means that let declarations do not move to the top of the current execution context.

**聽起來是因為 `var` 有 `hoisting` 而 `let` 沒有。**<br>

等等，你確定？看看剛才這個例子：

```
var a = 123;
if (true) {
  a = 456; // Uncaught ReferenceError: Cannot access 'a' before initialization at <anonymous>:3:7
  let a;
}
```

**注意：如果把 let 刪掉是不會報錯的**

```
var a = 123;
if (true) {
  a = 456; // ok
}
```

而且最詭異的點是它竟然報錯在第三行，如果 JS 是逐行執行的，那到第三行 `a = 456` 為止應該不會出錯才對。<br>

**可見 JS interpreter 跟我們想得不太一樣，要能解釋這個問題，我們只能理解 JS interpreter 有對 `let` 做一定程度的 `hoisting` ，不然沒道理在還沒執行到第四行以前就影響了第三行的賦值行為，那到底為什麼官方會否認？這背後又有什麼愛恨情仇糾葛不清呢？還請各位客倌先別急躁，敬待小弟下回分曉。**

**下集：**

- [回答一題 var / let / const 就驚艷你的面試官 ─ hoisting 細節篇](https://yuanwu0000.github.io/zachary-gitbook/articles/js/var-let-const/advanced.html)
