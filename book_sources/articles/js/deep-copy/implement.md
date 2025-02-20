# 面試官：前面講了這麼多，還是你直接實作一個深拷貝給我看看？

面試官：那些比較麻煩的資料型別可以暫時忽略，比如： `Symbol`, `Date`...為求簡化，`Array` 也暫時忽略好了，先單純考慮 `Object` 來實作。

我心理 OS：我可以用講的咪？我打字會緊張耶 QQ

### 吞一顆鎮定劑，讓我們開始吧！

首先，我們都知道以下這樣是行不通的：

```js
const obj = { a: 1 };
let copy = obj;
obj.a = 2; // copy.a 也會是 2
```

因為 copy 跟 obj **共用同一份記憶體位址。**

接著，我們試著來找出深拷貝的 base case：

```js
const obj = { a: 1 };
let copy = {};
copy.a = 1; // 複製成功
```

```js
const obj = { a: 1, b: 1 };
let copy = {};
copy.a = 1; // 複製成功
copy.b = 1; // 複製成功
```

因為我們需要把 `obj` 物件中的每個屬性都遍歷一遍，所以我們需要一個迴圈：

```js
const obj = { a: 1, b: 1 };
let copy = {};
for (let key in obj) {
  copy[key] = obj[key];
}
```

但這樣會遇到一個問題...

```js
const obj = { a: { aa: 1 } };
let copy = {};
for (let key in obj) {
  copy[key] = obj[key];
}
obj.a.aa = 2; // copy.a.aa 也會被改成 2
```

原因是這個例子裡我們**只複製了一層的物件**，而且沒有判斷 `obj.a` 的型別，所以如果 `obj.a` 同樣是物件的話，就又會遇到最開頭的問題。

可以發現，要**深度複製**一個物件，我們需要只能一層一層地把物件的**原始型別 (primitive)** 抓出來，並且遇到同樣是物件的屬性時，再重新"呼叫"一次，沒有錯，有感覺了嗎？通常這種情況就是該遞迴出場了：

```js
const obj = { a: { aa: 1 } };
function deepCopy(obj) {
  let obj_c = {};
  for (let key in obj) {
    // 判斷是物件的話則進入遞迴
    if (typeof obj[key] === "object") {
      obj_c[key] = deepCopy(obj[key]);
    } else {
      // 是原始型別則直接複製出來
      obj_c[key] = obj[key];
    }
  }
  return obj_c;
}
const copy = deepCopy(obj);
obj.a.aa = 2;
console.log(copy.a.aa); // 1
```

**如此一來你便完成的最基礎的深拷貝了！**
