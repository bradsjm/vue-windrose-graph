<script>
import Vue from "vue";
import WindRose from "@/WindRose.vue";

export default Vue.extend({
  name: "ServeDev",
  components: {
    WindRose,
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
      :value="[10, 5, 8, 3, 9, 4, 6, 10, 5, 3, 6, 8, 4, 9, 15, 12]"
    />
  </div>
</template>
