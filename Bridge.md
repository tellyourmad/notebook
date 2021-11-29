## 需求分析

### 技术背景

在使用 `iframe` 的项目中，宿主页面（后面简称 **Host**）和寄生页面（后面简称 **Parasite**）之间通过 `postMessage` 和 `window.addEventListener("message", fn)` 进行信息交换：

```javascript
// Host
window.removeEventListener("message", function (event) {
  window.iframe[0].postMessage(
    `你好，我是 Host，我已经收到你的“${event.data}”消息`
  );
});
```

```javascript
// Parasite
window.removeEventListener("message", function (event) {
  console.log(event.data);
});
window.parent.postMessage("你好，我是 Parasite，这是一条消息A");
window.parent.postMessage("你好，我是 Parasite，这是一条消息B");
```

### 面临问题

可以看到，上面例子是简单**监听**和**发送**，其存在两个明显的问题：

- 不可靠，`postMessage` 只管推送消息，而不在乎是否成功发送，没有“确认”流程
- 非线性，顺序触发多次 `postMessage` 后，接收端其实是不一定会以同样顺序接收到信息

### 期望效果

理想的效果应该是能让调用者以 `async/await` 的方式去发送消息

```javascript
// Parasite
const resultA = await send("消息A");
console.log(resultA); // 你好，我是 Host，我已经收到你的“消息A”消息
const resultB = await send("消息B");
console.log(resultB); // 你好，我是 Host，我已经收到你的“消息B”消息
```

目标就是要满足以下要求：

- 有求必应，有请求有响应
- 同步逻辑，编写符合直觉的、可读性高的代码
- 逻辑黑盒，调用者无需考虑其中过程，只要当调用异步方法一样使用即可

## 上机操作

### 角色分工

这里把逻辑划分成三份：

- 宿主（Host） ：只运行在宿主页面上的逻辑
- 寄生者（Parasite）：只运行在寄生页面上的逻辑
- 信使（Courier）：负责上面二者之间的信息传递

数据流动方式：

Host ↔ HostCourier ↔ ParasiteCourier ↔ Parasite

Host 和 Parasite 分别通过各自的 Courier 与对方进行通讯

### 信使 Courier
