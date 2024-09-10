# 關於深淺拷貝的前備知識：你知道什麼是 call by sharing 嗎？

身為前端工程師，我想你一定在面試時遇過這樣的問題：<br>

「請解釋深淺拷貝的不同？」<br>
「哦~就是有沒有共用記憶體的區別呀！」<br>
「那你覺得實務上什麼時候會出現淺拷貝？非原始型別在 assign 的時候會出現什麼行為？原理是什麼？」<br>
「Hmmm......」<br>

我認為這是一題可以從 junior 問到 senior 的問題，大部分的人都能答出差別在記憶體共用，但能詳細展開的人卻不多，我想這就是這題的鑑別度所在了。<br>

拒絕死背硬記，今天我們來揭開所有一知半解的觀念吧！工程師嘛！得時刻保持著求知欲才能不被時代淘汰。

### 一個簡單的範例：

```
let testA = { a: 1 }
let testB = testA
testB.a = 2
console.log(testA) // { a: 2 }
```

因為物件的 assignment 在 javascript 裡面等於將**記憶體位址** pass 給另一個變數，也是所謂的 **call by reference** (注意：這個用詞可能不夠**精確**，後面會提到)，所以 testB 跟 testA 實際上是**同一個物件**，更改 testB 的屬性當然也就會改到 testA。<br>

junior 程度的工程師能夠解釋到這裡基本上就過關了，但是，如果你想了解更多，請繼續往下看...

### 再一個簡單的範例：

```
let testA = { a: 1 }
let testB = 1
let testC = { c: 1 }
function change() {
  testA = { a: 2 }
  testB = 3
  testC.c = 2
}
change()
console.log(testA, testB, testC) // {a: 2} 3 {c: 2}
```

這邊我們在函數裡面**直接操作外部變數**，一切都是那麼符合預期，直到...

### 一個不那麼簡單的範例：

```
let testA = { a: 1 }
let testB = 1
let testC = { c: 1 }
function change(testA, testB, testC) {
  testA = { a: 2 }
  testB = 3
  testC.c = 2
}
change(testA, testB, testC)
console.log(testA, testB, testC) // {a: 1} 1 {c: 2}
```

注意到了嗎？當我們把外部變數當成參數傳遞進 `function` 的時候，會跟我們直接在函數內操作外部變數有很大的不同，而其中的關鍵點就在"**傳遞**"，javascript 的傳遞行為有貓膩！或是說，其中有我們了解不夠透徹的地方。

### 天竺取經

唐僧一行人為了尋求大乘佛法前往西方取經，過程極其艱辛。如今，我為了更加了解 Javascript 的實作原理，也決定遠赴電子絲路取經，只求能獲得真理致道。

你可能會想問，宣揚佛法有無上真經，那理解 javascript 又有什麼？我想，那就只能是**ECMA**了。

> ECMAScript 是一種由 Ecma 國際（前身為歐洲電腦製造商協會）在標準 ECMA-262 中定義的手稿語言規範。這種語言在全球資訊網上應用廣泛，它往往被稱為 JavaScript 或 JScript，但實際上後兩者是 ECMA-262 標準的實作和擴充。

與無上真經不同的是，ECMA **每年都會有所更新**，以下所提到的規範都會是 [ECMA-262 15th edition, June 2024](https://262.ecma-international.org/15.0/index.html#sec-intro) 的內容。

那麼就讓我們踏上旅途吧！

### 首先我好奇， javascript 是如何為變數賦值的

知曉了這個，便知道為什麼會有記憶體共用的情形產生。

我看到 ECMA 其中的[這個](https://262.ecma-international.org/15.0/index.html#sec-assignment-operators-runtime-semantics-evaluation)章節詳細定義了 javascript 如何處理 variable assignment。

<img src="../../../images/deep-copy-basics/assignment.PNG" width="1000" >

以前面的例子來說：
`let testA = { a: 1 }`

LeftHandSideExpression = `let testA`<br>
AssignmentExpression = `{ a: 1 }`
a. a. Let lref be ? Evaluation of LeftHandSideExpression.

### 形參 (Parameter) 與實參 (Argument)

形參相當於函數中定義的變數，調用函數傳遞參數的過程相當於定義形參變數並且用實參的值來初始化。

### References

http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/
https://github.com/mqyqingfeng/Blog/issues/7
https://github.com/mqyqingfeng/Blog/issues/10
