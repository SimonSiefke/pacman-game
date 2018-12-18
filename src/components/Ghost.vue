<!-- based on "Pacman and the ghosts" by Eoin Fanning (https://codepen.io/eoinf/pen/GYjNdY) -->
<template>
  <div class="ghost">
    <div class="eyes">
      <span class="l-eye" />
      <span class="r-eye" />
    </div>
    <div class="l" />
    <div class="r" />
  </div>
</template>

<style lang="stylus" scoped>
$base = (100 / 44) vmin

.ghost
  background currentcolor
  border-radius $base * (50 / 86) $base * (50 / 86) 0 0
  height $base
  margin-left 0.24vmin
  position absolute
  width $base * (76 / 86)

  &:before, &:after
    content ''

  &:before, &:after, .r, .l
    background currentColor
    border-radius 0 0 6px 6px
    bottom $base * (-12 / 86)
    height $base * (12 / 86)
    position absolute
    width $base * (16 / 86)

  .l
    left 0px

  .r
    right 0px

  &:before
    left $base * (20 / 86)

  &:after
    right $base * (20 / 86)

  // eyes
  .eyes
    display flex
    justify-content space-between
    left 50%
    position absolute
    top $base * (30 / 86)
    transform translateX(-50%)
    width 60%

    span
      background white
      border-radius 50%
      height $base * (16 / 86)
      position relative
      width $base * (16 / 86)

      &:before
        background #000
        border-radius $base * (4 / 86)
        content ''
        height $base * (8 / 86)
        left 0px
        position absolute
        top $base * (4 / 86)
        width $base * (8 / 86)
</style>
