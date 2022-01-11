# 「 春节游戏」过年的压岁钱，我真的不能收！

![](https://raw.githubusercontent.com/tzhiy/image-repo/main/blog/image-20220111145725922.png?token=APUDWWSQSD4ARMWXWHB7I33B3U3ZM)

>在线体验：[https://red-envelope.web.cloudendpoint.cn/](https://red-envelope.web.cloudendpoint.cn/)
>
>Github 地址：[https://github.com/tzhiy/Red-Envelope](https://github.com/tzhiy/Red-Envelope)

## 游戏背景

快过年了，大家在拜年的时候，压岁钱的习俗必不可少（虽然我已经过了领压岁钱的年龄）。每年在收到压岁钱的时候，我们都会推搡一下，可总是害怕推回去了。所以领压岁钱是一个技术活，这个游戏就模拟了领压岁钱的玩法，让我们在欲迎还拒的过程中最后领到压岁钱。

## 游戏玩法

在游戏中，我们需要扮演孩子的角色来应对亲戚的压岁钱。

![](https://raw.githubusercontent.com/tzhiy/image-repo/main/blog/image-20220111150543006.png?token=APUDWWWXTAQ4LFEDF2TGCJLB3U34I)

游戏开始后，屏幕下方的进度条会向左做随机速度的移动，我们每次点击屏幕，指针就会向右移动一段固定距离。

倒计时结束时，如果指针停留在有效区域，你就能获得红包了！

![](https://raw.githubusercontent.com/tzhiy/image-repo/main/blog/image-20220111150953725.png?token=APUDWWWXRLAHMKTQKPWJAQLB3U4AG)

如果指针在游戏过程中碰到进度条的边缘，那么游戏就会失败，压岁钱会缓缓离你而去。

![](https://raw.githubusercontent.com/tzhiy/image-repo/main/blog/image-20220111154632340.png?token=APUDWWS6KRDAYH2BENBZO3LB3U4BA)

## 代码说明

代码是使用 Vue 来写的，这里主要介绍一下进度条的指针处理，手部动画的处理和对话框的样式处理。

### 进度条的指针处理

首先初始化进度条，为了使有效区域在进度条中间部分，使用随机数生成有效区域的起始位置并根据起始位置得到终止位置。

```js
// 初始化进度条
const initVaildArea = () => {
  // ...
  // maxLength 是进度条的长度，length 是有效区域的长度
  pointDistance = maxLength / 2;
  length = 300;
  validStart = Math.random() * length - length + maxLength / 2;
  vaildArea._rawValue.style.left = validStart + "px";
  vaildArea._rawValue.style.width = length + "px";
};
```

玩家每次点击游戏区域，指针向右移动一段距离。

```js
// 玩家的操作
const handleProgress = () => {
  // 剧情结束后才能操作
  if (plotIsOver) {
    pointDistance += 320;
  }
};
```

每经过一段时间的间隔，指针向左移动一段距离，为了游戏的可玩性，设计了一个随机条件，触发后指针向左移动的距离增加。当倒计时未结束时触碰到进度条两端，游戏失败；结束后指针停在有效区，游戏成功，否则失败。

```js
// 初始化指针，倒计时
const initPointer = () => {
  pointer._rawValue.style.left = maxLength / 2 + "px";
  const timer = setInterval(() => {
    time.value -= 0.01;
    pointDistance -= 4;
    console.log(pointDistance);
    if (pointDistance <= 0 || pointDistance >= maxLength) {
      isLose.value = true;
      clearInterval(timer);
    }
    if (Math.random() < 0.2 && pointDistance - 6 >= 4) {
      pointDistance -= 6;
    }
    pointer._rawValue.style.left = pointDistance + "px";
    if (time.value < 0.01) {
      if (
        pointDistance >= validStart &&
        pointDistance <= validStart + length
      ) {
        isWin.value = true;
      } else {
        isLose.value = true;
      }
      clearInterval(timer);
    }
  }, 10);
};
```

### 两只手的处理

手的出现：

```js
// 手的出现
const initHands = () => {
  leftHand._rawValue.style.transform = "translateX(-200px)";
  rightHand._rawValue.style.transform = "translateX(200px)";
};
```

当游戏失败后，亲戚角色的手会缓慢撤回，撤回之前减慢过渡时间，撤回之后恢复原来的时间：

```js
// 撤回手
const removeHands = () => {
  rightHand._rawValue.style.transition = "transform 2s ease-in";
  rightHand._rawValue.style.transform = "translateX(800px)";
  setTimeout(() => {
    rightHand._rawValue.style.transition = "transform 0.5s ease-in";
  }, 2000);
};
```

手的移动距离是随机的，当游戏失败后，移动停止并撤回亲戚角色的手：

```js
const hangHands = () => {
  let leftHandDistance = -200;
  let rightHandDistance = 200;
  let time = 0;
  const timer = setInterval(() => {
    const random = Math.random() * 100 - 50;
    // 防止手移动到边框外
    if (leftHandDistance + random <= 0 && rightHandDistance + random >= 0) {
      leftHandDistance += random;
      rightHandDistance += random;
    }
    leftHand._rawValue.style.transform =
      "translateX(" + leftHandDistance + "px)";
    rightHand._rawValue.style.transform =
      "translateX(" + rightHandDistance + "px)";
    time++;
    if (time >= 60) {
      clearInterval(timer);
    }
  }, 250);
  watch(isLose, (newValue) => {
    if (newValue) {
      clearInterval(timer);
      removeHands();
    }
  });
};
```

### 对话框的样式

![](https://raw.githubusercontent.com/tzhiy/image-repo/main/blog/image-20220111154043496.png?token=APUDWWUBRZ3KHDUWSUZKLELB3U4AM)

对话框下方有一个三角部分，使用伪类实现，这里通过两个伪类的错开叠加实现边框的效果（展示的代码经过简化）：

```scss
.dialogs {
  position: relative;
  top: 80px;
  left: 0;
  right: 0;
  height: 160px;
  z-index: 1;
  &__left {
    display: inline-block;
    position: absolute;
    width: 240px;
    height: 110px;
    border-radius: 20px;
    border: 4px solid black;
    background-color: white;
    left: 20px;
  }
  &__left::after,
  &__left::before {
    position: absolute;
    width: 0;
    height: 0;
    content: "";
    left: 30px;
    top: 110px;
    border-right: 24px solid transparent;
    border-bottom: 16px solid transparent;
    border-top: 16px solid black;
    border-left: 24px solid black;
  }
  &__left::after {
    top: 104px;
    left: 34px;
    border-top: 16px solid white;
    border-left: 24px solid white;
  }
}
```

## 总结

这个游戏的创意来自于《中国式家长》这款游戏，我使用 Vue 将这个小游戏实现了下来，完整的代码可以查看 [Github](https://github.com/tzhiy/Red-Envelope)，欢迎提出建议。

