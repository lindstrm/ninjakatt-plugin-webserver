<template>
  <div class="log-container">
    <a href="#" @click.prevent="clearLog"> <i class="fas fa-trash"></i></a>
    <div v-if="entries.length === 0" style="padding-left: 10px">No log</div>
    <div v-for="(dayEntries, day) in entries" :key="day">
      <span>{{ day }}</span>
      <ul class="no-list">
        <li
          v-for="(entry, i) in dayEntries"
          :key="i"
          :class="'log-' + entry.color"
          :title="entry.date"
        >
          {{ entry.message.split(/\[.+?\]/)[1] }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { format } from "date-fns";

export default {
  name: "Logger",
  data() {
    return {
      inview: false,
      entries: [],
    };
  },
  mounted() {
    this.inview = true;
    this.logTimer();
  },

  beforeDestroy() {
    this.inview = false;
  },

  methods: {
    async logTimer() {
      const entries = await this.$http.get("/logger/log").then((res) =>
        res.data
          .split("\n")
          .filter((line) => line.length > 0)
          .map((line) => line.split(";"))
          .map((entry) => ({
            color: entry[0],
            date: format(new Date(entry[1]), "HH:mm"),
            message: entry[2],
            day: format(new Date(entry[1]), "EEE dd MMM"),
          }))
          .reverse()
      );

      this.entries = entries.reduce((prev, curr) => {
        if (!prev[curr.day]) {
          prev[curr.day] = [];
        }
        prev[curr.day] = [...prev[curr.day], curr];
        return prev;
      }, {});

      if (this.inview) {
        setTimeout(() => {
          this.logTimer();
        }, 10000);
      }
    },

    async clearLog() {
      await this.$http.delete("logger/log");
      this.entries = [];
      this.$forceUpdate();
      this.logTimer();
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../scss/_variables.scss";
.log-container {
  padding-top: 10px;
  max-height: 100vh;
  min-height: 100vh;
  font-size: 11px;
  overflow-y: auto;
  overflow-x: hidden;
  position: relative;
  a {
    position: absolute;
    top: 10px;
    right: 10px;
  }
  span {
    padding: 3px 5px;
    color: #fff;
  }
}
li {
  background-color: #111;
  padding: 3px 5px;
  max-width: 250px;
  white-space: normal;
  word-break: break-all;
  &:nth-child(odd) {
    background-color: #151515;
  }
}
.log-green {
  color: rgb(181, 235, 96);
}

.log-cyan {
  color: rgb(90, 206, 226);
}

.log-red {
  color: rgb(226, 90, 90);
}

.log-yellow {
  color: rgb(224, 226, 90);
}

.log-white {
  color: rgb(253, 253, 253);
}

.log-blue {
  color: rgb(117, 149, 235);
}
</style>
