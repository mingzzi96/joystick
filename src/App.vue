<template>
  <div id="app">
    <div id="joyStickWrap">
      <RangeSelector
        :min="0.1"
        :max="0.9"
        :isValueVisible="true"
        :step="0.05"
        @handleChangeInputValue="handleChangeInputValue"
      />
      <JoyStick
        :joyStickCustomOption="joyStickCustomOption"
        :joyStickReturnValue="joyStickReturnValue"
        :isJoyStickTouchActive="isJoyStickTouchActive"
        @handleJoyStickReturnValue="handleJoyStickReturnValue"
        @handleJoyStickEndMethod="handleJoyStickEndMethod"
        @handleJoyStickStartMethod="handleJoyStickStartMethod"
      />
    </div>
    <p>PITCH : {{ handyGimbalPitch }}</p>
    <p>YAW : {{ handyGimbalYaw }}</p>
  </div>
</template>

<script>
import JoyStick from "./components/JoyStick.vue";
import RangeSelector from "./components/RangeSelector.vue";

export default {
  name: "App",
  components: {
    JoyStick,
    RangeSelector,
  },
  data() {
    return {
      droneControl: [null, null, { droneId: this.moduleTypeIndex }, null, null],
      moduleTypeIndex: 2,
      handyShootingMode: "",
      handyGimbalPitch: 0,
      handyGimbalYaw: 0,
      handyDayNightTime: "",
      handyIntervalShootingOn: false,
      pitchMin: -20,
      pitchMax: 90,
      yawMin: -90,
      yawMax: 90,
      isJoyStickTouchActive: false,
      joyStickReturnValue: {},
      joyStickCustomOption: {
        mode: "static", // 'static' 또는 'dynamic'
        position: { left: "50%", top: "50%" }, // 초기 위치
        restOpacity: 1,
        follow: false,
        size: 150,
        lockX: false,
        lockY: false,
        color: "red",
      },
      joyStickSensitivity: 0.75,
    };
  },
  methods: {
    /**
     * 감도 직접 정할 수 있도록
     */
    handleChangeInputValue(sensitivity) {
      console.log(sensitivity);
      this.joyStickSensitivity = sensitivity;
    },
    /**
     * 조이스틱 터치 끝났을 때
     */
    handleJoyStickEndMethod() {
      this.isJoyStickTouchActive = false;

      if (this.joyStickInterval) {
        clearInterval(this.joyStickInterval);
        this.joyStickInterval = null;
      }
    },
    /**
     * 조이스틱 터치 시작되었을 때
     */
    handleJoyStickStartMethod() {
      this.isJoyStickTouchActive = true;
    },
    /**
     * 조이 스틱이 반지름 만큼 움직였을 경우 함수
     *
     */
    handleUpdateJoyStickPositionWhenTouched() {
      // 기존 타이머가 있으면 먼저 정리
      if (this.joyStickInterval) {
        clearInterval(this.joyStickInterval);
        this.joyStickInterval = null;
      }
      // 조이스틱 리턴값의 distance가 반지름과 같고 터치 상태라면?
      this.joyStickInterval = setInterval(() => {
        this.handyGimbalYaw +=
          Math.cos((this.joyStickReturnValue.angle.degree * Math.PI) / 180) *
          this.joyStickReturnValue.force *
          this.joyStickSensitivity;

        this.handyGimbalPitch +=
          Math.sin((this.joyStickReturnValue.angle.degree * Math.PI) / 180) *
          this.joyStickReturnValue.force *
          this.joyStickSensitivity;

        if (this.handyGimbalYaw > this.yawMax) {
          // yaw max 값 보다 커?
          this.handyGimbalYaw = this.yawMax;
        }

        if (this.handyGimbalYaw < this.yawMin) {
          // yaw min 값 보다 작아?
          this.handyGimbalYaw = this.yawMin;
        }

        if (this.handyGimbalPitch > this.pitchMax) {
          // pitchs max 값 보다 커?
          this.handyGimbalPitch = this.pitchMax;
        }

        if (this.handyGimbalPitch < this.pitchMin) {
          // pitchs min 값 보다 작아?
          this.handyGimbalPitch = this.pitchMin;
        }

        // vector값을 기존 값에 계속 더한 뒤 publish
        // this.droneControl[this.moduleTypeIndex].sendHandyGimbalDegree(
        //   this.handyGimbalYaw,
        //   this.handyGimbalPitch
        // );
      }, 10);
    },
    /**
     * 조이스틱 nipple.js 리턴 객체값 변수에 저장하고 publish 보내기
     */
    handleJoyStickReturnValue(data) {
      // joyStick을 움직였을 때에만 동작함
      if (this.droneControl[this.moduleTypeIndex]) {
        // return value 일단 변수에 저장
        this.joyStickReturnValue = data;

        if (
          this.joyStickReturnValue.distance ===
            this.joyStickCustomOption.size / 2 &&
          this.isJoyStickTouchActive
        ) {
          // 반지름만큼 움직였고, 터치 상태일 때
          console.log("반지름이야");
          this.handleUpdateJoyStickPositionWhenTouched();
        }

        if (
          this.joyStickReturnValue.distance !==
          this.joyStickCustomOption.size / 2
        ) {
          if (this.joyStickInterval) {
            clearInterval(this.joyStickInterval);
            this.joyStickInterval = null;
          }

          this.handyGimbalYaw +=
            this.joyStickReturnValue.vector.x * this.joyStickSensitivity;

          this.handyGimbalPitch +=
            this.joyStickReturnValue.vector.y * this.joyStickSensitivity;

          if (this.handyGimbalYaw > this.yawMax) {
            // yaw max 값 보다 커?
            this.handyGimbalYaw = this.yawMax;
          }

          if (this.handyGimbalYaw < this.yawMin) {
            // yaw min 값 보다 작아?
            this.handyGimbalYaw = this.yawMin;
          }

          if (this.handyGimbalPitch > this.pitchMax) {
            // pitchs max 값 보다 커?
            this.handyGimbalPitch = this.pitchMax;
          }

          if (this.handyGimbalPitch < this.pitchMin) {
            // pitchs min 값 보다 작아?
            this.handyGimbalPitch = this.pitchMin;
          }

          // vector값을 기존 값에 계속 더한 뒤 publish
          // this.droneControl[this.moduleTypeIndex].sendHandyGimbalDegree(
          //   this.handyGimbalYaw,
          //   this.handyGimbalPitch
          // );
        }
        return;
      }
      // return this.$store.commit(
      //   "openAlert",
      //   this.$t("droneAlert.pleaseSetDroneTypeAndId")
      // );
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#joyStickWrap {
  width: 100%;
  padding: 20px;
  min-height: 200px;
}
</style>
