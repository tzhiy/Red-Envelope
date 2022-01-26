<template>
  <div class="container">
    <main class="game-container" @click="handleProgress">
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
    </main>
    <header class="header">
      <div class="header__item header__link">
        <a href="https://github.com/tzhiy/Red-Envelope" target="_blank"
          >github</a
        >
      </div>
      <div class="header__item header__link">
        <a href="https://juejin.cn/post/7051861310526455838/" target="_blank"
          >掘金</a
        >
      </div>
      <div class="header__item header__audio" @click="handleAudio">
        {{ isAudio ? "🔊" : "🔈" }}
      </div>
      <div
        class="header__item header__difficulty"
        @click="handleChangeDifficulty"
      >
        {{ difficulty }}
      </div>
      <div class="header__item header__title">{{ titleContent }}</div>
      <div
        class="header__item header__help"
        @mouseenter="() => handleShowHelp(true)"
        @mouseleave="() => handleShowHelp(false)"
      >
        ?
      </div>
      <div class="help" v-show="isShowHelp">
        谁不想要压岁钱呢？🥰<br />
        1.
        游戏开始后，你需要通过点击鼠标操作下方进度条上的游标，倒计时结束时如果游标停留在橙色区域内，你就能获得红包了！<br />
        2. 点击左侧的难度标签可以切换难度<br />
        <p>1. 简单模式速度 * 1；</p>
        <p>2. 困难模式速度 * 1.6；</p>
        <p>
          3. 无尽模式速度 * 1，每次通关后速度随机增加 0.04 ~
          0.40，看看你能坚持到第几波~
        </p>
      </div>
      <div class="header__item header__start" @click="handleStart">
        开始游戏
      </div>
    </header>
    <audio
      src="http://dl.stream.qqmusic.qq.com/C4000042Ep7K2IXmF0.m4a?guid=1342169660&vkey=6ECCCC6B623EB23CC7EBFEB9EF7C8271D6EDFE30AD5999C34718D2D93D7CE7826CB7D20464833A8751ACCABFB927AB193712AFA7277D5D5C&uin=2570986081&fromtag=66"
      loop="true"
      ref="audioRef"
    ></audio>
  </div>
</template>

<script>
import { ref, watch } from "vue";

// 点击开始游戏，初始化所有内容
const useStartGameEffect = (time) => {
  const startProcess = ref(false);
  const handleStart = () => {
    startProcess.value = true;
    time.value = 15;
  };
  return { startProcess, handleStart };
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
  startProcess,
  difficulty,
  speedMuti,
  handleStart
) => {
  const startGameProcess = () => {
    showStartPlot();
  };

  const titleContent = ref("我真的不要压岁钱！");
  let successedTime = 0;
  watch(isShowPlot, (newValue) => {
    if (newValue === false) {
      if (difficulty.value === "无尽") {
        titleContent.value = `速度 * ${speedMuti.value.toFixed(
          2
        )}，已获得 🧧 ${successedTime} 个`;
      } else {
        titleContent.value = "我真的不要压岁钱！";
      }
      initHands();
      setTimeout(() => {
        hangHands();
        progress();
      }, 500);
    }
  });
  watch(isWin, (newValue) => {
    if (newValue) {
      if (difficulty.value === "无尽") {
        successedTime++;
        titleContent.value = `速度 * ${speedMuti.value.toFixed(
          2
        )}，已获得 🧧 ${successedTime} 个`;
        startProcess.value = false;
        speedMuti.value += Math.ceil(Math.random() / 0.1) * 0.04;
        handleStart();
        startGameProcess();
      } else {
        titleContent.value = "你获得了压岁钱！😋";
        startProcess.value = false;
      }
    }
  });
  watch(isLose, (newValue) => {
    if (newValue) {
      if (difficulty.value === "无尽") {
        titleContent.value = `游戏结束，最终获得 🧧 ${successedTime} 个！✌`;
        speedMuti.value = 1;
        successedTime = 0;
      } else {
        titleContent.value = "你失去了压岁钱！😭";
      }
      startProcess.value = false;
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

// 随机对话框
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
    "钱多钱少都是心意",
    "讨个好彩头，吉祥如意",
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
const useProgressEffect = (pointer, vaildArea, isShowPlot, speedMuti) => {
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
    length = 240;
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
  // 游标每秒的移动距离
  let pointDisPerSec = 3.6;
  // 玩家操作每秒移动的距离
  let pointDisPlayer = 140;
  watch(speedMuti, (newValue) => {
    pointDisPerSec = 3.6 * newValue;
    pointDisPlayer = 140 * newValue;
  });
  // 玩家的操作
  const handleProgress = () => {
    // 剧情结束后才能操作
    if (plotIsOver) {
      pointDistance += pointDisPlayer;
    }
  };
  // 初始化游标，倒计时
  const initPointer = () => {
    pointer._rawValue.style.left = maxLength / 2 + "px";
    const timer = setInterval(() => {
      time.value -= 0.01;
      pointDistance -= pointDisPerSec;
      if (pointDistance <= 0) {
        isLose.value = true;
        pointDistance = 0;
        clearInterval(timer);
      } else if (pointDistance >= maxLength) {
        isLose.value = true;
        pointDistance = maxLength - 10;
        clearInterval(timer);
      }
      if (Math.random() < 0.2 && pointDistance >= 300) {
        pointDistance -= 1.6;
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

// 难度设置
const useDifficultyEffect = () => {
  const difficultys = ["简单", "困难", "无尽"];
  const speedMutis = [1, 1.6, 1];
  const difficulty = ref("简单");
  const speedMuti = ref(1);
  let i = 0;
  const handleChangeDifficulty = () => {
    const index = ++i % difficultys.length;
    difficulty.value = difficultys[index];
    speedMuti.value = speedMutis[index];
  };
  return {
    difficulty,
    speedMuti,
    handleChangeDifficulty,
  };
};

// 音乐设置
const useAudioEffect = (audioRef) => {
  const isAudio = ref(false);
  const handleAudio = () => {
    if (isAudio.value) {
      audioRef.value.pause();
    } else {
      audioRef.value.play();
    }
    isAudio.value = !isAudio.value;
  };
  return {
    isAudio,
    handleAudio,
  };
};

export default {
  setup() {
    const leftHand = ref(null);
    const rightHand = ref(null);
    const pointer = ref(null);
    const vaildArea = ref(null);
    const audioRef = ref(null);
    const { isAudio, handleAudio } = useAudioEffect(audioRef);
    const { difficulty, speedMuti, handleChangeDifficulty } =
      useDifficultyEffect();
    const { isShowPlot, plotContent, handleHidePlot, showStartPlot } =
      useHeadPlotEffect();
    const { progress, isWin, isLose, handleProgress, time } = useProgressEffect(
      pointer,
      vaildArea,
      isShowPlot,
      speedMuti
    );
    const { initHands, hangHands } = useHandsEffect(
      leftHand,
      rightHand,
      isLose
    );
    const { myDialogContent, relativeDialogContent } =
      useDialogChangeEffect(time);
    const { isShowHelp, handleShowHelp } = useShowHelpEffect();
    const { startProcess, handleStart } = useStartGameEffect(time);
    const { startGameProcess, titleContent } = useGameProcessEffect(
      isShowPlot,
      showStartPlot,
      initHands,
      hangHands,
      progress,
      isWin,
      isLose,
      startProcess,
      difficulty,
      speedMuti,
      handleStart
    );
    watch(startProcess, (newValue) => {
      if (newValue) {
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
      difficulty,
      handleChangeDifficulty,
      isAudio,
      handleAudio,
      audioRef,
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

.header {
  position: absolute;
  display: flex;
  justify-content: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -368px);
  height: 60px;
  width: 1100px;
  background-color: rgb(250, 87, 37);
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
    flex: 2;
    margin-right: 10px;
  }
  &__audio {
    flex: 2;
  }
  &__difficulty {
    flex: 4;
    margin-left: 40px;
    background-color: rgb(255, 153, 0);
    cursor: pointer;
  }
  &__title {
    flex: 12;
    margin-left: 10px;
    margin-right: 10px;
    font-family: "Comic Sans MS";
  }
  &__help {
    flex: 2;
    margin-right: 60px;
  }
  &__start {
    flex: 8;
  }
  .help {
    position: absolute;
    width: 400px;
    left: 660px;
    top: 60px;
    background-color: rgb(255, 243, 136);
    text-align: left;
    font-size: 18px;
    border-radius: 12px;
    border: 2px solid black;
    box-sizing: border-box;
    padding: 4px 10px;
    line-height: 40px;
    p {
      padding: 0;
      margin: 0 0 0 16px;
      font-size: 16px;
      line-height: 32px;
    }
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

a {
  text-decoration: none;
  color: black;
}
</style>
