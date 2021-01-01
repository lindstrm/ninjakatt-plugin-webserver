<template>
  <div>
    <div class="box clearfix">
      <ul class="no-list" v-if="settings">
        <li><input type="text" v-model="newMovie" @keyup.enter="addMovie" placeholder="Add movie"></li>
      </ul>
      <div class="settings cursor-pointer" @click="settingsModal = true">
        <i class="fas fa-cogs"></i>
      </div>
    </div>
    <div class="container">
      <table class="datalist torrents" v-if="entries.length > 0">
        <thead>
          <th class="cursor-pointer">movie</th>
          <th class="cursor-pointer">torrents</th>
          <th class="cursor-pointer">
            downloaded
          </th>
          <th></th>
        </thead>
        <transition-group tag="tbody" name="list">
          <tr v-for="movie in entries" :class="movie.state" :key="movie.name">
            <td v-text="movie.name" class="cursor-pointer" @click="activeMovie = movie"></td>
            <td v-text="movie.torrents.length" class="cursor-pointer" @click="activeMovie = movie"></td>
            <td v-text="movie.torrents.map(x => x.downloaded).find(x => x === true) ? 'Yes' : 'No'" class="cursor-pointer" @click="activeMovie = movie"></td>
            <td>
              <span @click="removeMovie(movie)">
                <i class="fas fa-trash-alt cursor-pointer"></i>
              </span>
            </td>
          </tr>
        </transition-group>
      </table>
    </div>

    <vodal v-if="show" :show="show" @hide="show = false" :width="600" :height="400" :closeButton="false">
      <div v-if="activeMovie">
        <h2>{{ activeMovie.name }}</h2>
        <div class="torrent-container">
          <div class="torrent cursor-pointer" v-for="torrent in activeMovie.torrents" :key="torrent.fileName" @click="torrentToDownload = torrent" :class="{
            'selected': torrentToDownload && torrentToDownload.uri === torrent.uri
          }">
            {{ torrent.title }}
            <i class="fas fa-check green" v-if="torrent.downloaded"></i>
          </div>
        </div>
        <div class="download">
          <div class="save-path mb10">
            <input type="text" v-if="torrentToDownload" v-model="torrentToDownload.savePath" placeholder="Save path">
            <input type="text" v-else disabled placeholder="Save path">
          </div>
          <button class="btn" @click="downloadTorrent" :class="{ disabled: !!!torrentToDownload || (torrentToDownload && !torrentToDownload.savePath)  }">
            Download
          </button>
        </div>
      </div>
    </vodal>

    <vodal v-if="settingsModal" :show="settingsModal" @hide="settingsModal = false" :width="600" :height="300" :closeButton="false">
      <div style="height: 238px;">
        <h4>Save path</h4>
        <input type="text" placeholder="Save path" v-model="settings.savePath">
        <h4>Minimum resolution</h4>
        <select v-model="settings.minResolution">
          <option value></option>
          <option
            v-for="res in settings.validResolutions"
            :key="res"
            :value="res"
            v-text="res"
          ></option>
        </select>
      </div>
      <div class="center-text">
        <button class="btn" @click="saveSettings">
          Save
        </button>
      </div>
    </vodal>
  </div>
</template>

<script>
import { format } from "date-fns";

export default {
  name: "Movies",
  data() {
    return {
      inview: false,
      entries: [],
      settings: null,
      activeMovie: null,
      show: false,
      torrentToDownload: null,
      newMovie: null,
      settingsModal: false,
    };
  },
  watch: {
    activeMovie(movie) {
      this.show = !!movie;
    },
    show(state) {
      if (!state) {
        this.torrentToDownload = null;
        this.activeMovie = null;
      }
    },
  },
  mounted() {
    this.inview = true;
    this.moviesTimer();
  },

  beforeDestroy() {
    this.inview = false;
  },

  methods: {
    async moviesTimer() {
      const data = await this.$http.get("/movies").then((res) => res.data);
      this.setSettings(data);
      if (this.inview) {
        setTimeout(() => {
          this.moviesTimer();
        }, 10000);
      }
    },

    async downloadTorrent() {
      const data = await this.$http
        .post("movies/download", this.torrentToDownload)
        .then((res) => res.data);
      this.setSettings(data);
    },

    async addMovie() {
      const data = await this.$http
        .post("movies", { movie: this.newMovie })
        .then((res) => res.data);
      this.setSettings(data);
    },

    async removeMovie(movie) {
      const data = await this.$http
        .post("movies/remove", { movie: movie.name })
        .then((res) => res.data);
      this.setSettings(data);
    },

    async saveSettings() {
      const settings = this.settings;
      delete settings.validResolutions;
      const data = await this.$http
        .post("movies/settings", settings)
        .then((res) => res.data);
      this.setSettings(data);
      this.settingsModal = false;
    },

    setSettings(settings) {
      this.entries = settings.movies;
      delete settings.movies;
      this.settings = settings;

      this.entries.forEach((x) => {
        x.torrents.forEach((y) => {
          if (y.savePath.length) {
            return;
          }
          y.savePath = this.settings.savePath;
        });
      });
    },
  },
};
</script>

<style lang="scss">
@import "../scss/_variables.scss";
.box {
  clear: both;
  ul {
    margin: 0;
  }
}
.torrent-container {
  height: 255px;
  max-height: 255px;
  overflow: auto;
}
.torrent {
  padding: 5px 10px;
  border-radius: 3px;

  &.selected {
    background-color: lighten($background-color, 10%);
  }
}

.settings li {
  display: inline-block;
  margin-right: 10px;
  &:last-of-type {
    margin-right: 0;
  }
}
</style>
