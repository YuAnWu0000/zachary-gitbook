# 面試官：前面講了這麼多，還是你直接實作一個深拷貝給我看看？

面試官：那些比較麻煩的資料型別可以暫時忽略，比如： `Symbol`, `Date`...
為求簡化，`Array` 也暫時忽略好了，先單純考慮 `Object` 就可以了。

我心理 OS：我可以用講的咪？我打字會緊張耶 QQ

### 吞一顆鎮定劑，讓我們開始吧！

首先，我們都知道以下這樣是行不通的：

```js
const test = { a: 1 };
let copy = test;
test.a = 2; // copy.a 也會是 2
```

因為 copy 跟 test **共用同一份記憶體位址。**

接著，我們試著來找出深拷貝的 base case：

```js
const test = { a: 1 };
let copy;
copy[a] = 1; // 複製成功
```

```js
const test = { a: 1, b: 1 };
let copy;
copy[a] = 1; // 複製成功
copy[b] = 1; // 複製成功
```

因為我們需要把`test`物件中的每個屬性都遍歷一遍，所以我們需要一個迴圈：

```js
const test = { a: 1, b: 1 };
let copy;
for (let key in test) {
  copy[key] = test[key];
}
```

但這樣會遇到一個問題...

```js
const test = { a: { aa: 1 } };
let copy;
for (let key in test) {
  copy[key] = test[key];
}
test.a.aa = 2; // copy.a.aa 也會被改成 2
```

可以發現，要"深度複製"一個物件，只能一層一層的把物件的原始型別抓出來，
並且遇到同樣是物件的屬性時，再重新"呼叫"一次的感覺，沒有錯，遇到這種感覺就是該遞迴出場了：
