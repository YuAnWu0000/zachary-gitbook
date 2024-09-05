# 深淺拷貝的前備知識：你知道什麼是 call by sharing 嗎？

身為前端工程師，我想你一定在面試時這樣的問題：<br>

「請解釋深淺拷貝的不同？」<br>
「哦...就是 call by refrence 跟 call by value 的區別呀！」<br>
「那你可以詳細說明一下各自的定義以及跟深淺拷貝的關係嗎？」<br>
「Hmmm......」<br>

我認為這是一題可以從 junior 問到 senior 的問題，大部分的人都能答出 call by refrence, call by value，但能詳細展開的人卻不多，這就是這題的鑑別度所在了。<br>

不如我們來把揭開所有一知半解的部分吧！工程師嘛！時刻保持著求知欲才能不被時代淘汰。

### 一個簡單的範例：

```
let testA = { a: 1 }
let testB = testA
testB.a = 2
console.log(testA) // { a: 2 }
```

因為物件的 assignment 在 javascript 裡面等於將記憶體位址 pass 給另一個變數，也是所謂的 call by reference，所以 testB 跟 testA 實際上是同一個物件，更改 testB 的屬性當然也就會改到 testA 了。<br>

junior 程度的工程師能夠解釋到這裡基本上就過關了，但是，如果你想了解更多，請繼續往下看...

### 一個不那麼簡單的範例：

```
let testA = { a: 1 }
let testB = 1

```
