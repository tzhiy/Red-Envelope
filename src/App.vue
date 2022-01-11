<template>
  <div class="container">
    <div class="game-container" @click="handleProgress">
      <div class="time">
        <span class="time__content">
          {{ time.toFixed(2) }}
        </span>
      </div>
      <div class="dialogs">
        <div class="dialog dialogs__left">
          <div class="dialog__content">{{ myDialogContent }}</div>
        </div>
        <div class="dialog dialogs__right">
          <div class="dialog__content">{{ relativeDialogContent }}</div>
        </div>
      </div>
      <div class="hands">
        <div class="hand hands__left" ref="leftHand" />
        <div class="hand hands__right" ref="rightHand" />
      </div>
      <div class="progress">
        <div class="progress__bar">
          <div class="progress__pointer" ref="pointer"></div>
          <div class="progress__vaild-area" ref="vaildArea"></div>
        </div>
      </div>
      <div class="plot" v-show="isShowPlot" @click="handleHidePlot">
        {{ plotContent }}
      </div>
    </div>
    <div class="aside">
      <div class="aside__item aside__link">
        <a href="https://github.com/tzhiy/Red-Envelope" target="_blank"
          >github</a
        >
      </div>
      <div class="aside__item aside__link">
        <a href="https://juejin.cn/post/7051861310526455838/" target="_blank"
          >掘金</a
        >
      </div>
      <div class="aside__item aside__title">{{ titleContent }}</div>
      <div
        class="aside__item aside__help"
        @mouseenter="() => handleShowHelp(true)"
        @mouseleave="() => handleShowHelp(false)"
      >
        ?
      </div>
      <div class="help" v-show="isShowHelp">
        谁不想要压岁钱呢？🥰<br />
        游戏开始后，点击屏幕操作进度条上的指针，倒计时结束时如果指针停留在有效区域，你就能获得红包了！
      </div>
      <div class="aside__item aside__start" @click="handleStart">开始游戏</div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from "vue";

// 点击开始游戏，初始化所有内容
const useStartGameEffect = (startProcess, time) => {
  const handleStart = () => {
    startProcess.value = true;
    time.value = 15;
  };
  return { handleStart };
};

// 开始游戏流程
const useGameProcessEffect = (
  isShowPlot,
  showStartPlot,
  initHands,
  hangHands,
  progress,
  isWin,
  isLose,
  startProcess
) => {
  const startGameProcess = () => {
    showStartPlot();
  };

  const titleContent = ref("我真的不要压岁钱！");

  watch(isShowPlot, (newValue) => {
    if (newValue === false) {
      titleContent.value = "我真的不要压岁钱！";
      initHands();
      setTimeout(() => {
        hangHands();
        progress();
      }, 500);
      watch(isWin, (newValue) => {
        if (newValue) {
          titleContent.value = "你获得了压岁钱！😋";
          startProcess.value = false;
        }
      });
      watch(isLose, (newValue) => {
        if (newValue) {
          titleContent.value = "你失去了压岁钱！😭";
          startProcess.value = false;
        }
      });
    }
  });
  return { startGameProcess, titleContent };
};

// 开始的对话框
const useHeadPlotEffect = () => {
  let isShowPlot = ref(false);
  const momPlot = ref([
    "妈妈：来，给你三舅磕个头！小时候对你可好了~你都忘了？",
    "妈妈：来，给你二叔磕个头！开心起来啊~",
    "妈妈：来，给你大姨磕个头！从小最亲你了~",
    "妈妈：来，给你小舅母磕个头！可得礼貌点哈~",
    "妈妈：来，给你舅母磕个头！小时候尿布都是她帮你换的，你不记得了~",
  ]);
  let plotContent = ref("内容");
  const handleHidePlot = () => {
    isShowPlot.value = false;
  };
  const showStartPlot = () => {
    isShowPlot.value = true;
    plotContent.value =
      momPlot.value[Math.floor(Math.random() * momPlot.value.length)];
  };
  return { plotContent, handleHidePlot, showStartPlot, isShowPlot };
};

// 手的操作
const useHandsEffect = (leftHand, rightHand, isLose) => {
  // 手的出现
  const initHands = () => {
    leftHand._rawValue.style.transform = "translateX(-200px)";
    rightHand._rawValue.style.transform = "translateX(200px)";
  };
  // 撤回手
  const removeHands = () => {
    rightHand._rawValue.style.transition = "transform 2s ease-in";
    rightHand._rawValue.style.transform = "translateX(800px)";
    setTimeout(() => {
      rightHand._rawValue.style.transition = "transform 0.5s ease-in";
    }, 2000);
  };
  const hangHands = () => {
    let leftHandDistance = -200;
    let rightHandDistance = 200;
    let time = 0;
    const timer = setInterval(() => {
      const random = Math.random() * 100 - 50;
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
  return { initHands, hangHands };
};

// 用 time 来随机对话框
const useDialogChangeEffect = (time) => {
  const myDialogList = ref([
    "使不得使不得！",
    "我都这么大了，不该收了",
    "都不容易，这钱我怎么能要",
    "平时帮我太多，不收不收",
    "别这样，别这样",
    "这样太不好意思了，不行！",
    "心意收下了，钱就算了",
  ]);
  const relativeDialogList = ref([
    "快拿着快拿着！",
    "你不收就是看不起我！",
    "哎，别客气，都是自己人",
    "不用看你妈脸色，收下吧！",
    "收下收下，应该的嘛",
    "压岁习俗不能省！",
  ]);
  const myDialogContent = ref("使不得使不得！");
  const relativeDialogContent = ref("快拿着快拿着！");
  watch(time, () => {
    // time 每 0.01s 改变一次
    if (Math.random() < 0.002) {
      const myDialogRandom = Math.floor(
        Math.random() * myDialogList.value.length
      );
      myDialogContent.value = myDialogList.value[myDialogRandom];
    }
  });
  watch(time, () => {
    // time 每 0.01s 改变一次
    if (Math.random() < 0.002) {
      const relativeDialogRandom = Math.floor(
        Math.random() * relativeDialogList.value.length
      );
      relativeDialogContent.value =
        relativeDialogList.value[relativeDialogRandom];
    }
  });
  return { myDialogContent, relativeDialogContent };
};

// 进度条的操作
const useProgressEffect = (pointer, vaildArea, isShowPlot) => {
  const maxLength = 968;
  const isWin = ref(false);
  const isLose = ref(false);
  let pointDistance;
  let length;
  let validStart;
  let time = ref(15);
  // 初始化进度条
  const initVaildArea = () => {
    isWin.value = false;
    isLose.value = false;
    pointDistance = maxLength / 2;
    length = 300;
    validStart = Math.random() * length - length + maxLength / 2;
    vaildArea._rawValue.style.left = validStart + "px";
    vaildArea._rawValue.style.width = length + "px";
  };
  let plotIsOver = false;
  watch(isShowPlot, (newValue) => {
    if (newValue === false) {
      plotIsOver = true;
    }
  });
  // 玩家的操作
  const handleProgress = () => {
    // 剧情结束后才能操作
    if (plotIsOver) {
      pointDistance += 320;
    }
  };
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
  const progress = () => {
    initVaildArea();
    initPointer();
  };
  return { progress, isWin, isLose, handleProgress, time };
};

// 显示帮助
const useShowHelpEffect = () => {
  const isShowHelp = ref(false);
  const handleShowHelp = (sign) => {
    isShowHelp.value = sign;
  };
  return { isShowHelp, handleShowHelp };
};

export default {
  setup() {
    const leftHand = ref(null);
    const rightHand = ref(null);
    const pointer = ref(null);
    const vaildArea = ref(null);
    let startProcess = ref(false);
    const { isShowPlot, plotContent, handleHidePlot, showStartPlot } =
      useHeadPlotEffect();
    const { progress, isWin, isLose, handleProgress, time } = useProgressEffect(
      pointer,
      vaildArea,
      isShowPlot
    );
    const { initHands, hangHands } = useHandsEffect(
      leftHand,
      rightHand,
      isLose
    );
    const { myDialogContent, relativeDialogContent } =
      useDialogChangeEffect(time);
    const { isShowHelp, handleShowHelp } = useShowHelpEffect();
    const { startGameProcess, titleContent } = useGameProcessEffect(
      isShowPlot,
      showStartPlot,
      initHands,
      hangHands,
      progress,
      isWin,
      isLose,
      startProcess
    );
    const { handleStart } = useStartGameEffect(startProcess, time);
    watch(startProcess, (newValue) => {
      if (newValue === true) {
        startGameProcess();
      }
    });

    return {
      time,
      isShowPlot,
      handleStart,
      plotContent,
      showStartPlot,
      handleHidePlot,
      leftHand,
      rightHand,
      pointer,
      vaildArea,
      titleContent,
      handleProgress,
      myDialogContent,
      relativeDialogContent,
      isShowHelp,
      handleShowHelp,
    };
  },
};
</script>

<style lang='scss'>
body {
  background: linear-gradient(
    to right,
    rgb(230, 78, 17),
    rgb(238, 115, 0),
    rgb(248, 112, 0)
  );
}

.game-container {
  position: absolute;
  top: 50%;
  left: 50%;
  background: linear-gradient(
    to right,
    rgb(240, 69, 1),
    rgb(231, 209, 5),
    rgb(248, 112, 0)
  );
  transform: translate(-50%, -50%);
  width: 1100px;
  height: 600px;
  border: 4px solid black;
  user-select: none;
}

.time {
  position: absolute;
  left: 0;
  right: 0;
  top: 10px;
  height: 60px;
  text-align: center;
  &__content {
    display: inline-block;
    line-height: 60px;
    background-color: black;
    color: white;
    border-radius: 20px;
    font-family: "Comic Sans MS";
    font-size: 24px;
    font-weight: 700;
    width: 100px;
  }
}

.progress {
  position: absolute;
  bottom: 30px;
  left: 0;
  right: 0;
  height: 40px;
  &__bar {
    position: absolute;
    background-color: rgb(230, 230, 165);
    left: 60px;
    right: 60px;
    height: 28px;
    border-radius: 16px;
    border: 6px solid rgb(243, 139, 41);
  }
  &__pointer {
    position: absolute;
    left: 484px;
    width: 10px;
    top: -6px;
    height: 40px;
    background-color: black;
    transition: left 0.05s linear;
    z-index: 1;
  }
  &__vaild-area {
    position: absolute;
    left: 0;
    top: 0;
    height: 28px;
    background-color: rgb(238, 182, 0);
  }
}

.dialogs {
  position: relative;
  top: 80px;
  left: 0;
  right: 0;
  height: 160px;
  z-index: 1;
  .dialog {
    display: inline-block;
    position: absolute;
    width: 240px;
    height: 110px;
    border-radius: 20px;
    border: 4px solid black;
    background-color: white;
    &__content {
      position: absolute;
      left: 20px;
      right: 20px;
      bottom: 21px;
      top: 21px;
      font-size: 24px;
      font-weight: bold;
      line-height: 34px;
    }
  }
  &__left {
    left: 20px;
  }
  &__right {
    right: 20px;
  }
  .dialog::after,
  .dialog::before {
    position: absolute;
    width: 0;
    height: 0;
    content: "";
  }
  &__left::after,
  &__left::before {
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
  &__right::after,
  &__right::before {
    right: 30px;
    top: 110px;
    border-left: 24px solid transparent;
    border-bottom: 16px solid transparent;
    border-top: 16px solid black;
    border-right: 24px solid black;
  }
  &__right::after {
    top: 104px;
    right: 34px;
    border-top: 16px solid white;
    border-right: 24px solid white;
  }
}

.plot {
  box-sizing: border-box;
  position: absolute;
  bottom: 20px;
  left: 20px;
  right: 20px;
  background-color: rgb(255, 187, 109);
  z-index: 4;
  height: 220px;
  border-radius: 12px;
  border: 4px solid rgb(117, 64, 4);
  padding: 30px 40px;
  line-height: 40px;
  font-size: 20px;
  font-weight: bold;
}

.aside {
  position: absolute;
  display: flex;
  justify-content: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -368px);
  height: 60px;
  width: 1100px;
  background-color: rgb(235, 92, 49);
  border: 4px solid black;
  font-weight: bold;
  line-height: 48px;
  font-size: 20px;
  text-align: center;
  user-select: none;
  &__item {
    border-radius: 8px;
    margin: 2px 0;
    border: 4px solid black;
    background-color: rgb(255, 230, 0);
  }
  &__link {
    flex: 1;
    margin-right: 20px;
  }
  &__title {
    flex: 7;
    margin-left: 180px;
    margin-right: 10px;
    font-family: "Comic Sans MS";
  }
  &__help {
    flex: 1;
    margin-right: 20px;
  }
  &__start {
    flex: 4;
  }
}

.hands {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 80px;
  height: 400px;
  overflow: hidden;
  .hand {
    position: absolute;
    width: 800px;
    height: 360px;
    bottom: 20px;
    transition: transform 0.5s ease-in;
    background-size: contain;
    background-repeat: no-repeat;
  }
  &__left {
    left: 0;
    transform: translateX(-800px);
    background-image: url("./assets/lefthand.png");
    background-position: right;
    z-index: 2;
  }
  &__right {
    right: 0;
    transform: translateX(800px);
    background-image: url("./assets/righthand.png");
    background-position: left;
    z-index: 3;
  }
}

.help {
  position: absolute;
  width: 300px;
  left: 660px;
  top: 60px;
  background-color: rgb(255, 243, 136);
  text-align: left;
  font-size: 18px;
  border-radius: 12px;
  border: 2px solid black;
  box-sizing: border-box;
  padding: 4px 10px;
}

a {
  text-decoration: none;
  color: black;
}
</style>
