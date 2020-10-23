<script>
import Vue from "vue";
import WindRose from "@/WindRose.vue";

export default Vue.extend({
  name: "ServeDev",
  components: {
    WindRose,
  },
  data() {
    return {
      windrose: [10, 5, 8, 3, 9, 4, 6, 15, 5, 3, 6, 8, 4, 9, 15, 12],
      odometer: 0,
      interval: undefined,
    };
  },
  mounted() {
    this.interval = setInterval(() => {
      this.odometer += Math.random() * 5;
      this.windrose.push(this.windrose.shift());
    }, 5000);
  },
});
</script>

<template>
  <div id="app">
    <WindRose
      size="201"
      title="Wind Rose"
      backgroundColor="BEIGE"
      foregroundType="TYPE1"
      frameDesign="TILTED_GRAY"
      odometerVisible="true"
      :odometerValue="odometer"
      :value="windrose"
    />
  </div>
</template>
