# async/await 在 forEach 裡面不起作用？你是否搞錯了什麼

![meme](../../images/forEach-async-await/meme.jpg)

### 「ㄟ~Zachary~~你可以過來幫我通靈一下嗎？」

前幾天公司同事遇到了 forEach 與 async/await，卡關了許久仍不明所以，
於是叫上我幫忙解惑，身為通靈王的我，盯著他如下的程式碼：

```
const basket = [
  {
    name: 'apple',
    number: 1
  },
  {
    name: 'banana',
    number: 2
  },
  {
    name: 'mango',
    number: 3
  }
]
async function buyEachOne() { // 希望拿到每種水果+1的結果
  basket.forEach(async (item) => {
    const newNumber = await shopping(item.number)
    item.number = newNumber
  })
  console.log(basket)
}
function shopping(number) {
  return new Promise((resolve, reject) => {
    setTimeout(function() {
      resolve(number + 1)
    }, 2000)
  });
}
buyEachOne()
```

> 我將概念梳理了一下，流程簡化為菜籃中有若干種水果，我想為每樣水果都多添購一顆，每次購買須花費兩秒的時間，購買完成後將菜籃中所有水果的新數量打印出來。

好的！回到問題...<br>

「到底為什麼 console 出來數量沒有變啊！」他一臉懊惱的抱著頭低鳴<br>
<img src="../../images/forEach-async-await/console.jpg" width="268" height="110">

<!-- ![console](../../images/forEach-async-await/console.jpg) -->

「而且...點了一下箭頭...」<br>
<img src="../../images/forEach-async-await/console_expand.jpg" width="176" height="268">

<!-- ![console](../../images/forEach-async-await/console_expand.jpg) -->

「你看！我要的值又長出來啦！是怎樣，為什麼這世界這麼複雜啊啊啊？」<br>

見他開始歇斯底里，同樣也深感困惑的我開始在網路上找尋相關資料，發現會造成這種現象其實可以歸咎於兩個原因，也就是說，造成我同事崩潰的犯人，有兩位！<br>

**Chrome DevTool, Async/Await**<br>

就是你們(指

> ### 釋迦牟尼《法句經》中有言：莫輕小惡，以為無罪，小惡所積，足以滅身。

### Chrome DevTool

我們先說行小惡的兇手，Chrome DevTool！沒錯，就是你，不要以為把頭撇開我就抓不到你了(Chrome:
關我屁事。)<br>
事實上只要你把鼠標移到上圖的藍色 icon，就會出現 tooltip 告訴你原因了："Value
below was evaluated just now."<br>
意思是：以 chrome 來說，展開前的數值是 console 當下的值，而展開後則是現在存放在記憶體中最新的值，這其實是 chrome 的一個小巧思，讓開發者能夠更好地追蹤該地址的值前後為何，只是在這個例子中不幸變成苦惱我同事的一個盲點。

至於為什麼 console 當下數值會沒變更呢？這問題就牽涉到了我們今天的千古罪人，async/await 了！(async/await: 你再繼續誣陷我兩兄弟，我就把你打到你媽都不認識你。)

> 其實他倆兄弟是無辜的，前面只是玩笑話而已，各位客官請別太認真，他們現在拿著球棒站在我身後呢！

### async / await

我們都知道 JavaScript 是單執行緒的語言，其內部有著 Event Loop 機制，各位可能很困惑，奇怪，怎麼被威脅了一下，就開始講其他的東西不講 async/await 了呢？除了他們倆個手上的球棒真的很大根之外，其實，這些都是環環相扣的，觀念缺一不可。

如果對 Event Loop 機制不理解的人，可以參照<a href="https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html">這篇</a>文章

**文章真的非常重要**<br>
**文章真的非常重要**<br>
**文章真的非常重要**<br>

我決定等你看完再繼續講解，放心，我很閒， 我可以在這等你一天都不是問題。<br>
至於如何算是完全理解 Event Loop？ 對不起，可能要請你去問 Brendan Eich，不過...<br>

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/YOo001UM8PI/0.jpg)](https://www.youtube.com/watch?v=YOo001UM8PI)

當你可以完全預測上面這個影片的走向時，我相信你就可以繼續閱讀以下的文章了。

<a href="http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D">範例</a>來源

### 讓我們用上面影片的邏輯來重新檢視一下這個範例

```
async function buyEachOne() { // 希望拿到每種水果+1的結果
  basket.forEach(async (item) => {
    const newNumber = await shopping(item.number)
    item.number = newNumber
  })
  console.log(basket)
}
function shopping(number) {
  return new Promise((resolve, reject) => {
    setTimeout(function() {
      resolve(number + 1)
    }, 2000)
  })
}
```

我們首先觀察 `buyEachOne()` 這個 function，程式先用 `forEach()` 對 `basket` 進行遍歷，依序執行傳入 `forEach()` 的三個 callback，執行到 `shopping()` 的時候發現遇到了 `setTimeOut()`，因此將 `setTimeOut()` 交由 Web APIs 進行處理，接著由於 `await` 的效果，暫時凍結內部執行環境，至此，第一個 callback 處理完畢。**這樣的流程會重複不間斷地執行三次。**

`forEach()` 的三個 callback "處理"完以後 (**注意：此時三個 callback 內部仍然是凍結狀態，因為在等待`setTimeout()`**)，接著程式執行到了 `console.log()` 這一行，所以才有了上面 chrome 的打印：**水果們並沒有得到+1 的結果**。

> 試想如果單執行緒的 JS 沒有 Web API 輔助，停留在原地等待三個 callback 執行完畢，那總共就是等待六秒的時間才能執行其他任務，網頁如果阻塞六秒，使用者早就已經跑光光了 :P
