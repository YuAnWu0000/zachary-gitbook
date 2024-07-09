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
];
async function buyEachOne() { // 希望拿到每種水果+1的結果
  basket.forEach(async (item) => {
    const newNumber = await shopping(item.number);
    item.number = newNumber;
  });
  console.log(basket);
}
function shopping(number) {
  return new Promise((resolve, reject) => {
    setTimeout(function() {
      resolve(number + 1);
    }, 2000);
  });
}
buyEachOne();
```

> 我將概念梳理了一下，流程簡化為菜籃中有若干種水果，我想為每樣水果都多添購一顆，每次購買須花費兩秒的時間，購買完成後將菜籃中所有水果的新數量打印出來。

好的！回到問題...<br>

「到底為什麼 console 出來是下面這樣啊！」他一臉懊惱的抱著頭嘶吼<br>
![console](../../images/forEach-async-await/console.jpg)

「而且...點了一下箭頭...」<br>
![console](../../images/forEach-async-await/console_expand.jpg)

「你看！我要的值又長出來啦！是怎樣，為什麼這世界這麼複雜啊啊啊？」<br>

見他開始抱著頭歇斯底里，同樣也深感困惑的我開始在網路上找尋相關資料，發現會造成這種現象其實可以歸咎於兩個原因，也就是說，造成我同事崩潰的犯人，有兩位！<br>

**Chrome dev tool, Async/Await**<br>
就是你們(指

> **釋迦牟尼《法句經》中有言：莫輕小惡，以為無罪，小惡所積，足以滅身。**

我們先說行小惡的兇手，Chrome dev tool！沒錯，就是你，不要以為把頭撇開我就抓不到你了(Chrome:
關我屁事。)<br>
事實上只要你把鼠標移到上圖的藍色 icon，就會出現 tooltip 告訴你原因了："Value
below was evaluated just now."<br>
意思是：以 chrome 來說，展開前的數值是 console 當下的值，而展開後則是現在存放在記憶體中最新的值，
至於兩者有什麼不同？為什麼 console 當下數值會還沒變更呢？這問題就牽涉到了我們今天的千古罪人，
Async/Await 了！(Async/Await: 你再繼續誣陷我兩兄弟，我就把你打到你媽都不認識你。)
