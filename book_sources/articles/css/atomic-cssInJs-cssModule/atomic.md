# CSS Framework 到底怎麼選？Tailwind css ? styled components ? CSS Module ? (2)

前一篇講到，假設我們有一個接收三個 props 的 button component，以 CSS module 的方式會這樣做:

```
// jsx
export function MyButton({ variant, size, status }) {
  return
    <div className={`button button-${variant} button-${size} button-${status}`>
      Button
    </div>
}
// how to use
<MyButton variant="primary" size="big" status="disable"/>
```

```
/* sass */
.button {
  width: 120px;
  height: 30px;
  fontSize: 14px;
  fontWeight: 500;
  color: white;
  background-color: black;
  &-primary: {
    font-weight: bold;
    background-color: green;
    border-radius: 10px;
  }
  &-big {
    width: 200px;
    height: 50px;
  }
  &-disable {
    pointer-events: none;
  }
}
```

### 前一篇提到 css-in-js 可以利用 **Object 後蓋前**的特性來處理, 那麼 Atomic css 又要如何將組件的 props 跟 style 綁定呢？

```
import { twMerge } from 'tailwind-merge'
const MyButton = ({ variant, size, isDisable }) => {
  const defaultStyles =
    'w-[120px] h-[30px] text-base font-medium text-white bg-black border border-solid'
  const variantStyles = {
    primary: 'bg-green-500 font-medium border-white rounded-lg',
    warning: 'bg-orange-500 font-medium border-white rounded-lg',
    error: 'bg-red-500 font-medium border-white rounded-lg'
  }
  const sizeStyles = {
    big: 'w-[200px] h-[50px] text-lg',
    small: 'w-[100px] h-[20px] text-sm'
  }
  const disableStyles = 'bg-gray-500 pointer-events-none'
  let totalStyles = twMerge(
    defaultStyles,
    variantStyles[variant] ? variantStyles[variant] : '',
    sizeStyles[size] ? sizeStyles[size] : '',
    isDisable ? disableStyles : ''
  )
  return <div className={totalStyles}>Button</div>
}
export default MyButton
// how to use
<MyButton variant="primary" size="big" isDisable={false} />
```

答案就是要使用 `tailwind-merge` 套件來做合併。<br>
原因是對 Tailwind 來說一切都是字串，因此我們在合併的時候**沒辦法像 css-in-js 利用 Object 的屬性後蓋前，也沒辦法利用原生 CSS class 的後蓋前**，在這種限制之下，只能使用外部套件來幫我們合併 style。

**儘管如此，Tailwind CSS 仍然擁有最短的程式碼。**

### 效能比較

https://juejin.cn/post/7257711054901624869
