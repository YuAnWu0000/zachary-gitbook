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

因為我們需要把 `obj` 物件中的每個屬性都遍歷一遍，顯然我們需要一個迴圈：

```js
const obj = { a: 1, b: 1 };
let copy = {};
for (let key in obj) {
  copy[key] = obj[key];
}
obj.a = 2; // copy.a 依舊是 1, 複製成功
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

原因是這個例子裡我們**只複製了一層的物件**，而且沒有判斷 `obj.a` 的型別，所以如果 `obj.a` 同樣是物件的話，就又會遇到最開頭共用記憶體位址的問題。

可以發現，要**深度複製**一個物件，我們需要一層一層地把物件的**原始型別 (primitive)** 抓出來，當遇到物件時，再重新"呼叫"函式一次，沒有錯，有感覺了嗎？通常這種情況就是該遞迴出場了：

```js
const obj = { a: { aa: 1 } };
function deepCopy(obj) {
  let obj_c = {};
  for (let key in obj) {
    if (typeof obj[key] === "object") {
      // 判斷是物件的話則開啟遞迴
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
console.log(copy.a.aa); // 不受影響依舊是 1, 深度複製成功
```

**如此一來你便完成的最基礎的深拷貝了！**

### 面試官：接下來可以請你加上陣列的擴充嗎？

沒問題，讓我再吞一顆鎮定劑...<br>
開玩笑的！其實有了前面的 prototype 之後，擴充陣列其實只需要加一個判斷就可以搞定：

```js
const arr = [{ a: 1 }, 2, 3];
function deepCopy(obj) {
  let obj_c = Array.isArray(obj) ? [] : {}; // 加入此行
  for (let key in obj) {
    if (typeof obj[key] === "object") {
      obj_c[key] = deepCopy(obj[key]);
    } else {
      obj_c[key] = obj[key];
    }
  }
  return obj_c;
}
const copy = deepCopy(arr);
arr[0].a = 2;
console.log(copy[0].a); // 不受影響依舊是 1, 深度複製成功
```

咦？怎麼比想像中的簡單？有兩個原因：

1. 由於 javascript 原生並沒有`array`型別，因此`typeof [] === 'object'`，無須多寫一個`else if`。
2. `for...in` 這個語法可以同時適用於 `object` & `array`，ex:
   ```js
   for (let key in ["a", "b", "c"]) {
     console.log(key); // 印出index: 0, 1, 2
   }
   ```
   好！到這你已經完成陣列的擴充了。

### 面試官：你考慮過哪些 edge case 了呢？

比方說，上面的程式傳入一些奇怪的參數...

```js
console.log(deepCopy(null)); // {}
console.log(deepCopy("abc")); // { 0: 'a', 1: 'b', 2: 'c' }
console.log(deepCopy(1)); // {}
```

可以看到連字串都被強制轉成物件了，原理其實跟前面 `array` 有點像：

```js
for (let key in "abc") {
  console.log(key); // 印出index: 0, 1, 2
}
```

結論就是我們的 type check 做的不夠確實，因此還需要多加一行判斷：

```js
function deepCopy(obj) {
  if (!obj || typeof obj !== "object") return obj; // 加入此行
  let obj_c = Array.isArray(obj) ? [] : {};
  for (let key in obj) {
    if (typeof obj[key] === "object") {
      obj_c[key] = deepCopy(obj[key]);
    } else {
      obj_c[key] = obj[key];
    }
  }
  return obj_c;
}
console.log(deepCopy(null)); // null
console.log(deepCopy("abc")); // abc
console.log(deepCopy(1)); // 1
```

注意這邊可千萬不能只有判斷 `typeof obj !== "object"`，因為 `typeof null`其實也是 `object`，這會導致 `null` 並沒有 early return 回來，而是繼續進到後面的步驟。

**到這邊，相信你的面試已經過關了**，如果他還堅持要考 `new Date()` 跟 `Symbol()` 的處理的話，那我只能說...

塊陶鴨~
