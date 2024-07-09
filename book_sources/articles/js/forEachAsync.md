# async/await 在 forEach 裡面不起作用？我覺得你可能搞錯了什麼

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
「而且...點了一下箭頭...」<br>
「你看！我要的值又長出來啦！是怎樣，為什麼這世界這麼複雜啊啊啊？」<br>

見他開始抱著頭歇斯底里，同樣也深感困惑的我開始在網路上找尋相關資料，發現會造成這種現象其實可以歸咎於兩個原因，也就是說，造成我同事崩潰的犯人，有兩位！<br>

**Chrome dev tool, Async/Await**<br>
就是你們(指
