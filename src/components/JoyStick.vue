<template>
  <div id="joystick_container" ref="joyStickContainer">
    <div class="joystick_box" ref="joyStickBox"></div>
  </div>
</template>
<script>
import nipplejs from "nipplejs";

export default {
  name: "JoyStick",
  emits: ["handleJoyStickReturnValue", "handleJoyStickEndMethod"],
  props: {
    joyStickCustomOption: {
      type: Object,
      require: true,
    },
    joyStickReturnValue: {
      type: Object,
      require: false,
    },
    isJoyStickTouchActive: {
      type: Boolean,
      require: false,
    },
  },
  data() {
    return {
      isLoading: true,
      joyStickOption: this.joyStickCustomOption,
      joystick: null,
      // currentPosition: { x: 0, y: 0 }, // 현재 위치
      storedAngle: null, // 저장된 각도
      storedForce: null, // 저장된 힘
    };
  },
  methods: {
    handleStart(evt, data) {
      this.$emit("handleJoyStickStartMethod");
      // 초기 각도와 힘 저장
      this.storedAngle = data.angle ? data.angle.degree : null;
      this.storedForce = data.force;
    },
    handleMove(evt, data) {
      this.$emit("handleJoyStickReturnValue", data);
    },
    handleEnd() {
      this.$emit("handleJoyStickEndMethod");
    },
    createJoyStick(option) {
      this.joystick = nipplejs.create(option);

      this.joystick.on("start", this.handleStart);
      this.joystick.on("move", this.handleMove);
      this.joystick.on("end", this.handleEnd);
    },
  },
  mounted() {
    this.joyStickOption.zone = this.$refs.joyStickBox;

    // 이벤트 등록
    this.createJoyStick(this.joyStickOption);
  },
  beforeUnmount() {
    // 이벤트 및 타이머 정리
    if (this.joystick) {
      this.joystick.destroy();
    }
  },
};
</script>
<style scoped>
#joystick_container {
  position: relative;
  height: 90%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.joystick_box {
  position: relative;
  /* height: 90%;
  width: 90%; */
  width: 300px;
  height: 300px;
}
</style>
