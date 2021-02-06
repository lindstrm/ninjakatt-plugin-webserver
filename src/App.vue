<template>
  <div id="app" :class="{ minimized: minimenu }">
    <main-menu @minimize="minid => (minimenu = minid)"></main-menu>
    <div class="content-container">
      <div class="content">
        <router-view></router-view>
      </div>
      <div class="logger">
        <logger></logger>
      </div>
    </div>
  </div>
</template>

<script>
import MainMenu from "./components/MainMenu.vue";
import Logger from "./components/Logger.vue";
export default {
  components: { MainMenu, Logger },
  data() {
    return {
      minimenu: false,
      loggerInstalled: false,
    };
  },
  mounted() {
    this.loggerInstalled = !!this.$api.plugins.find((x) => x.name === "logger");
  },
};
</script>

<style lang="scss">
@import "./scss/_variables.scss";

#app {
  height: 100vh;
  .content-container {
    padding-left: 140px;
    display: grid;
    height: 100vh;
    max-height: 100vh;
    grid-template-columns: 1fr 250px;
    .content {
      overflow: auto;
    }
    .logger {
      background-color: darken($background-color, 5%);
    }
  }

  &.minimized {
    .content-container {
      padding-left: 50px;
    }
  }
}
</style>
