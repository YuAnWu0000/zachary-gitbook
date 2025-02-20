# 面試官：前面講了這麼多，還是你直接實作一個深拷貝給我看看？

面試官：那些比較麻煩的資料型別可以暫時忽略，比如： `Symbol`, `Date`...
為求簡化，`Array` 也暫時忽略好了，先單純考慮 `Object` 就可以了。

我心理 OS：還是我用講的？我打字會緊張耶 QQ

### 吞一顆鎮定劑，讓我們開始吧！

首先，我們都知道以下這樣是行不通的：

```js
const test = { a: 1 };
const copy = test;
test.a = 2; // copy.a 也會是 2
```

因為 copy 跟 test **共用同一份記憶體位址。**

接著，我們試著來找出深拷貝的 base case：

```js
const test = { a: 1 }
const copy
copy[a] = 1 // 複製成功
```

```js
const test = { a: 1, b: 1 }
const copy
copy[a] = 1 // 複製成功
copy[b] = 1 // 複製成功
```

可以發現，我們需要把`test`物件中的每個屬性都遍歷一遍，因此我們需要一個迴圈：

```js
const test = { a: 1, b: 1 }
const copy
for (ley key in test) {
  copy[key] = test[key]
}
```

```js
const test = { a: { aa: 1 } }
const copy
copy[a] = test.a
```

可以發現，要完整"複製"一個物件，需要把物件
