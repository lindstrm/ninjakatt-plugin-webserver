<template>
  <ul class="context-menu" v-show="show">
    <slot></slot>
  </ul>
</template>

<script>
export default {
  props: {
    toggle: {
      required: true,
      type: HTMLElement,
    },
    show: {
      type: Boolean,
    },
  },
  mounted() {
    setTimeout(() => console.log(this.toggle.getBoundingClientRect()), 1000);
  },
};
</script>

<style lang="scss" scoped>
@import "../scss/_variables.scss";

.context-menu {
  width: 100px;
  position: absolute;
  background-color: lighten($background-color, 5%);
  list-style: none;
}
</style>
