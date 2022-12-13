<template>
  <div class="image-container" ref="container">
    <img
      :style="leftImageStyle"
      alt="add alt"
      src="../assets/left.png"
      class="left-image"
      ref="leftImage"
    />

    <img
      alt="add alt"
      src="../assets/right.png"
      class="right-image"
      ref="rightImage"
    />

    <div :style="sliderStyle" class="slider">
      <div :style="lineStyle" class="line" />
      <div :style="handleStyle" class="handle">
        <div :style="leftArrowStyle" class="left-arrow" />
        <div :style="rightArrowStyle" class="right-arrow" />
      </div>
      <div :style="lineStyle" class="line" />
    </div>
  </div>
</template>

<script>
import { ResizeSensor } from "css-element-queries";

const defaultImageWidth = 0;
const defaultHandleSize = 40;

export default {
  name: "CompareImage",
  data() {
    return {
      positionPct: 0.5,
      imageWidth: defaultImageWidth,
      handleSize: defaultHandleSize,
      sliderLineWidth: 2,
    };
  },
  computed: {
    leftImageStyle() {
      return {
        clip: `rect(auto, ${this.imageWidth * this.positionPct}px, auto, auto)`,
      };
    },
    sliderStyle() {
      return {
        cursor: "ew-resize",
        left: this.imageWidth * this.positionPct - this.handleSize / 2 + "px",
        width: `${this.handleSize}px`,
      };
    },
    lineStyle() {
      return {
        width: `${this.sliderLineWidth}px`,
      };
    },
    handleStyle() {
      return {
        border: `${this.sliderLineWidth}px solid white`,
        height: `${this.handleSize}px`,
        width: `${this.handleSize}px`,
      };
    },
    leftArrowStyle() {
      return {
        border: `inset ${this.handleSize * 0.15}px rgba(0,0,0,0)`,
        borderRight: `${this.handleSize * 0.15}px solid white`,
        marginLeft: `-${this.handleSize * 0.25}px`,
        marginRight: `${this.handleSize * 0.25}px`,
      };
    },
    rightArrowStyle() {
      return {
        border: `inset ${this.handleSize * 0.15}px rgba(0,0,0,0)`,
        borderLeft: `${this.handleSize * 0.15}px solid white`,
        marginRight: `-${this.handleSize * 0.25}px`, // for IE11
      };
    },
  },
  methods: {
    setImageWidth() {
      // @ts-ignore
      this.imageWidth = this.$refs.rightImage.getBoundingClientRect().width;
    },
    startSliding(e) {
      // Prevent default behavior other than mobile scrolling
      if (!("touches" in e)) {
        e.preventDefault();
      }
      // Slide the image even if you just click or tap (not drag)
      this.updateSliderPosition(e);
      window.addEventListener("mousemove", this.updateSliderPosition);
      window.addEventListener("touchmove", this.updateSliderPosition);
    },
    finishSliding() {
      window.removeEventListener("mousemove", this.updateSliderPosition);
      window.removeEventListener("touchmove", this.updateSliderPosition);
    },
    updateSliderPosition(event) {
      const e = event || window.event;
      const cursorXfromViewport = e.touches ? e.touches[0].pageX : e.pageX;
      const cursorXfromWindow = cursorXfromViewport - window.pageXOffset;
      const imagePosition = this.$refs.rightImage.getBoundingClientRect();
      let pos = cursorXfromWindow - imagePosition.left;
      const minPos = this.sliderLineWidth / 2;
      const maxPos = this.imageWidth - this.sliderLineWidth / 2;
      if (pos < minPos) pos = minPos;
      if (pos > maxPos) pos = maxPos;
      this.positionPct = pos / this.imageWidth;
    },
  },
  mounted() {
    new ResizeSensor(this.$refs.container, () => {
      this.setImageWidth();
    });
    const containerElement = this.$refs.container;
    // for mobile
    containerElement.addEventListener("touchstart", this.startSliding);
    window.addEventListener("touchend", this.finishSliding);
    // for desktop
    if (this.hover) {
      containerElement.addEventListener("mouseenter", this.startSliding);
      containerElement.addEventListener("mouseleave", this.finishSliding);
    } else {
      containerElement.addEventListener("mousedown", this.startSliding);
      window.addEventListener("mouseup", this.finishSliding);
    }
  },
  beforeUnmount() {
    this.finishSliding();
    window.removeEventListener("mouseup", this.finishSliding);
    window.removeEventListener("touchend", this.finishSliding);
  },
};
</script>

<style lang="scss" scoped>
.image-container {
  box-sizing: border-box;
  position: relative;
  width: 100%;
  overflow: hidden;
}
.left-image {
  display: block;
  height: 100%; // fit to the height of under image
  object-fit: cover; // protrudes is hidden
  position: absolute;
  top: 0;
  z-index: 2;
  width: 100%;
}
.right-image {
  display: block;
  height: auto;
  width: 100%;
}
.slider {
  align-items: center;
  display: flex;
  flex-direction: column;
  height: 100%;
  justify-content: center;
  position: absolute;
  z-index: 3;
  top: 0;
  .line {
    background: white;
    box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2),
      0px 2px 2px 0px rgba(0, 0, 0, 0.14), 0px 1px 5px 0px rgba(0, 0, 0, 0.12);
    flex: 0 1 auto;
    height: 100%;
  }
  .handle {
    align-items: center;
    border-radius: 100%;
    box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2),
      0px 2px 2px 0px rgba(0, 0, 0, 0.14), 0px 1px 5px 0px rgba(0, 0, 0, 0.12);
    box-sizing: border-box;
    display: flex;
    flex: 1 0 auto;
    justify-content: center;
  }
  .left-arrow {
    height: 0;
    width: 0;
  }
  .right-arrow {
    height: 0;
    width: 0;
  }
}
</style>
