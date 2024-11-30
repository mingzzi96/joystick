<template>
  <div>
    <label v-if="label" :for="id">{{ label }}</label>
    <input
      type="range"
      :name="name"
      :id="id"
      :min="min"
      :max="max"
      :v-model="rangeValue"
      :step="step"
      @input="changeInputRangeValue"
    />
    <div class="input_number_container">
      <button
        v-if="hasButtonController"
        class="down"
        @click="clickDecreaseNumber"
      >
        &#45;
      </button>
      <div class="input_box">
        <div v-if="isValueVisible">{{ rangeValue }}</div>
        <input
          v-if="isValueEditInputVisible"
          type="number"
          :name="name"
          :id="id"
          :v-model="rangeValue"
          :min="min"
          :max="max"
          @input="changeInputNumberValue"
        />
        &#176;
      </div>
      <button
        v-if="hasButtonController"
        class="up"
        @click="clickIncreaseNumber"
      >
        &#43;
      </button>
    </div>
  </div>
</template>
<script>
export default {
  name: "RangeSelector",
  emits: [
    "handleClickIncreaseNumberValue",
    "handleClickDecreaseNumberValue",
    "handleChangeRangeBarValue",
    "handleChangeInputNumberValue",
    "handleChangeInputValue",
  ],
  props: {
    name: {
      type: String || Number,
      required: false,
      default: "range",
    },
    id: {
      type: String || Number,
      required: false,
      default: "input-range",
    },
    min: {
      type: Number,
      required: true,
    },
    max: {
      type: Number,
      required: true,
    },
    step: {
      type: Number,
      required: false,
      default: 1,
    },
    label: {
      type: String,
      required: false,
    },
    isValueVisible: {
      type: Boolean,
      required: false,
      default: false,
    },
    isValueEditInputVisible: {
      type: Boolean,
      required: false,
      default: false,
    },
    hasButtonController: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      // changeNumberValue: 0,
      // changeRangeValue: 0,
      rangeValue: this.min,
    };
  },
  methods: {
    /**
     * range를 움직여서 value 셋팅하는 경우의 처리
     */
    changeInputRangeValue(event) {
      if (this.checkMinMaxValue()) {
        this.rangeValue = Number(event.target.value);
        console.log(this.rangeValue);
        this.$emit("handleChangeRangeBarValue");
        this.$emit("handleChangeInputValue", this.rangeValue);
      }
    },
    /**
     * 숫자를 직접 셋팅해서 value 셋팅하는 경우의 처리
     */
    changeInputNumberValue() {
      if (this.checkMinMaxValue()) {
        this.rangeValue = Number(event.target.value);
        console.log(this.rangeValue);
        this.$emit("handleChangeInputNumberValue");
        this.$emit("handleChangeInputValue", this.rangeValue);
      }
    },
    /**
     * value 값을 step만큼 감소 시킨다.
     */
    clickDecreaseNumber() {
      if (!this.isLowerThanMinValue()) {
        this.rangeValue = this.rangeValue - this.step;
        this.$emit("handleClickDecreaseNumberValue");
        this.$emit("handleChangeInputValue", this.rangeValue); // handyGimbalPitch 업데이트
      }
    },
    /**
     * value 값을 step만큼 증가 시킨다..
     */
    clickIncreaseNumber() {
      if (!this.isHigherThanMaxValue()) {
        this.rangeValue = this.rangeValue + this.step;
        this.$emit("handleClickIncreaseNumberValue");
        this.$emit("handleChangeInputValue", this.rangeValue); // handyGimbalPitch 업데이트
      }
    },
    /**
     * min/max 값을 넘거나 부족할 경우 알럿을 띄워준다.
     */
    checkMinMaxValue() {
      if (this.rangeValue > this.max) {
        this.rangeValue = this.max;
        alert(this.max + "보다 크면 안돼");
        return false;
      }

      if (this.rangeValue < this.min) {
        this.rangeValue = this.min;
        alert(this.min + "보다 작으면 안돼");
        return false;
      }

      return true;
    },
    /**
     * min 값보다 낮아질 때 true 띄우고 알럿 띄우기 아니면 false
     */
    isLowerThanMinValue() {
      if (this.rangeValue <= this.min) {
        this.rangeValue = this.min;
        alert(this.min + "보다 작으면 안돼");
        return true;
      }

      return false;
    },
    /**
     * max 값보다 낮아질 때 true 띄우고 알럿 띄우기 아니면 false
     */
    isHigherThanMaxValue() {
      if (this.rangeValue >= this.max) {
        this.rangeValue = this.max;
        alert(this.max + "보다 크면 안돼");
        return true;
      }

      return false;
    },
  },
};
</script>
<style></style>
