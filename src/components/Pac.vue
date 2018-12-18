<template>
  <div :style="pacStyle">
    <PacEating v-if="state==='eating'" :class="`hat-${hatColor}`"/>
    <PacNormal v-else-if="state==='normal'"  :class="`hat-${hatColor}`"/>
  </div>
</template>

<script>
import PacEating from "@/components/PacIcons/PacEating.vue";
import PacNormal from "@/components/PacIcons/PacNormal.vue";
export default {
  components: {
    PacNormal,
    PacEating
  },
  props: {
    state: {
      type: String,
      required: true,
      validator(value) {
        return ["normal", "eating"].includes(value);
      }
    },
    direction: {
      type: String,
      required: true,
      validator(value) {
        return ["left", "right", "up", "down", "none"].includes(value);
      }
    },
    hatColor: {
      type: String,
      required: true,
      validator(value) {
        return ["red", "blue", "green", "black", "rainbow"].includes(value);
      }
    }
  },
  computed: {
    pacStyle() {
      switch (this.direction) {
        case "left":
          return { transform: "scaleX(-1)" };
        case "down":
          return { transform: "rotate(90deg)" };
        case "up":
          return { transform: "rotate(-90deg)" };
        default:
          return {};
      }
    }
  }
};
</script>


<style lang="stylus" scoped>
$base = (100 / 44) vmin

svg
  max-height 100%
  max-width 100%
  position absolute

  &.hat-red
    --hat-color #9b1517

  &.hat-green
    --hat-color green

  &.hat-blue
    --hat-color blue

  &.hat-black
    --hat-color black
</style>
