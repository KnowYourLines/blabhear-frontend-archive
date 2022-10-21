<template>
  <div>
  </div>
</template>
<script>
export default {
  name: "StopWatch",
  props: {
    running: {
      type: Boolean,
      default: false,
    },
    resetWhenStart: {
      type: Boolean,
      default: false,
    },
    restWhenStop: {
      type: Boolean,
      default: false,
    },
  },
  watch: {
    running: function (newVal) {
      if (newVal) this.startT();
      else this.stopT();
    },
  },
  mounted() {
    if (this.running) this.startT();
  },
  data() {
    return { seconds: 0, timer: null };
  },
  methods: {
    stopT: function () {
      clearInterval(this.timer);
      if (this.restWhenStop) this.resetT();
    },
    startT: function () {
      if (this.resetWhenStart) this.resetT();
      this.timer = setInterval(() => {
        this.seconds++;
        this.$emit("second-passed")
      }, 1000);
    },
    resetT: function () {
      this.seconds = 0;
    },
  },
};
</script>

<style scoped>
p {
  font-weight: bold;
  font-size: x-large;
}
</style>
