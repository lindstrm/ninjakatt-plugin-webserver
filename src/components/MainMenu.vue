<template>
  <nav>
    <router-link to="/" class="logo"></router-link>
    <ul class="inline-block">
      <li v-for="(plugin, k) in pluginsWithRoute" :key="k">
        <router-link :to="plugin">
          <span v-text="plugin"></span>
        </router-link>
      </li>
      <li>
        <router-link :to="{ name: 'versions' }">
          <span>versions</span>
        </router-link>
      </li>
    </ul>
    <div class="toggle-size" @click="toggle()">
      <i class="fas" :class="minimized ? 'fa-angle-double-right' : 'fa-angle-double-left'"></i>
    </div>
  </nav>
</template>

<script>
import { sortString } from "../helpers/sort.js";
export default {
  data() {
    return {
      minimized: false,
    };
  },
  mounted() {
    this.minimized = localStorage.getItem("minimized")
      ? JSON.parse(localStorage.getItem("minimized"))
      : false;
    this.toggle(true);
  },
  methods: {
    toggle(justEmit = false) {
      if (!justEmit) {
        this.minimized = !this.minimized;
        localStorage["minimized"] = this.minimized;
      }
      this.$emit("minimize", this.minimized);
    },
  },
  computed: {
    pluginsWithRoute() {
      const activeRoutes = this.$router.options.routes.map((r) => r.name);
      return this.$api.plugins
        .map((p) => p.name)
        .filter((p) => activeRoutes.indexOf(p) > -1)
        .sort(sortString);
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../scss/_variables.scss";

a.logo {
  text-decoration: none;
  margin-bottom: 10px;
  height: 30px;
  width: 100%;
  color: red;

  &:before {
    background-color: transparent;
    padding: 10px;
    word-wrap: none;
    display: block;
    white-space: nowrap;
    content: "(˚̨ ˚ )~~~~~~";
  }
}

nav {
  background-color: $nav-bg;
  position: fixed;
  height: 100vh;
  width: 140px;
  z-index: 10;
  box-shadow: 9px 0px 45px -11px rgba(0, 0, 0, 0.75);
  ul {
    position: relative;
    margin: 0;
    padding: 0;
    list-style: none;
    width: 100%;
    line-height: 0;
  }
  li a {
    line-height: normal;
    padding: 10px;
    color: $text-color;

    &:hover {
      span {
        color: $nav-bg;
      }
      &:before {
        width: 100%;
        background-color: lighten($nav-bg, 50%);
      }
    }
  }

  .toggle-size {
    position: fixed;
    bottom: 10px;
    left: 18px;
    cursor: pointer;

    i {
      color: $hover-color;
    }
  }
}

a {
  display: block;
  position: relative;
  span {
    display: inline-block;
    z-index: 1;
    position: relative;
  }
}

a::before {
  content: "";
  display: block;
  width: 0;
  height: 100%;
  top: 0;
  left: 0;
  position: absolute;
  background: lighten($nav-bg, 5%);
  transition: all 0.3s;
  margin: 0 auto;
  z-index: 0;
}

a.router-link-exact-active:not(.logo) {
  color: $nav-bg;
}

a.router-link-exact-active:not(.logo)::before {
  width: 100%;
  background-color: lighten($nav-bg, 20%);
}
</style>

<style>
#app.minimized nav {
  width: 50px;
}
#app.minimized a.logo::before {
  content: "^.^";
}

#app.minimized nav a {
  padding: 10px 10px 5px 10px;
}

#app.minimized nav a span {
  width: 24px;
  padding-left: 10px;
  letter-spacing: 10px;
  overflow: hidden;
  text-transform: capitalize;
}
</style>
