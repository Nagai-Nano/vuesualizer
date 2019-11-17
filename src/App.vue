<template>
  <div class="w-100 h-100 position-relative">
    <div class="blurred-background w-100 h-100 position-absolute" />
    <div class="content w-100 h-100 position-absolute">
      <div class="container">
        <canvas ref="canvas" :class="{ 'd-none': loading }" class="w-100" />
        <template v-if="!loading">
          <div class="song-progress">
            <div class="h-100 bg-light" :style="{ width: progress }"></div>
          </div>
          <div class="d-flex">
            <img
              class="thumbnail border border-dark rounded shadow-sm mr-2"
              :src="`/thumbnails/${thumbnails[index]}`"
              alt="thumbnail"
            />
            <div
              class="text-uppercase text-truncate d-flex flex-column justify-content-center text-light"
            >
              <p class="song-name m-0 font-weight-bold text-truncate">{{ songInfo.name }}</p>
              <span class="h4 text-truncate">{{ songInfo.author }}</span>
            </div>
          </div>
        </template>
        <Spinner v-else />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Spinner from "./Spinner";

export default {
  components: {
    Spinner
  },
  data() {
    return {
      loading: true,
      songs: [],
      thumbnails: [],
      index: 0,
      audio: new Audio(),
      progress: "0%"
    };
  },
  methods: {
    async fetchData() {
      const paths = ["/songs", "/thumbnails"];
      const promises = await Promise.all(paths.map(axios.get));
      this.songs = [...promises[0].data];
      this.thumbnails = [...promises[1].data];
    },

    initPlayer() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      const { width, height } = canvas;

      this.playSong();
      const context = new AudioContext();
      const analyser = context.createAnalyser();
      const src = context.createMediaElementSource(this.audio);

      src.connect(analyser);
      analyser.connect(context.destination);
      analyser.fftSize = 512;
      analyser.smoothingTimeConstant = 0.8;

      renderFrame();
      function renderFrame() {
        requestAnimationFrame(renderFrame);
        const dataArray = new Uint8Array(analyser.frequencyBinCount);
        analyser.getByteFrequencyData(dataArray);

        ctx.clearRect(0, 0, width, height);
        ctx.fillStyle = "#fff";

        for (let i = 0, len = dataArray.length; i < len; i++) {
          let barX = i * 7;
          let barWidth = 6;
          let barHeight = -(dataArray[i] / 2.5);
          ctx.fillRect(barX, height, barWidth, barHeight);
        }
      }
    },

    playSong() {
      this.audio.src = `/songs/${this.songs[this.index]}`;
      this.audio.play();
    },

    updateProgress() {
      const newTime = this.audio.currentTime * (100 / this.audio.duration);
      this.progress = newTime + "%";
    },

    nextSong() {
      this.index = (this.index + 1) % this.songs.length;
      this.playSong();
    }
  },
  computed: {
    songInfo() {
      const re = /\.(mp3|aac|ogg|m4a|wma|flac|wav)$/i;
      const [name, author] = this.songs[this.index].replace(re, "").split("-");
      return { name, author };
    }
  },
  async created() {
    try {
      await this.fetchData();
      this.initPlayer();
      this.audio.addEventListener("timeupdate", this.updateProgress);
      this.audio.addEventListener("ended", this.nextSong);
    } catch {
      throw new Error("Opps! Something went wrong");
    } finally {
      this.loading = false;
    }
  },
  beforeDestroy() {
    this.audio.removeEventListener("timeupdate", this.updateProgress);
    this.audio.removeEventListener("ended", this.nextSong);
  }
};
</script>

<style>
body {
  line-height: 1;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  font-family: "Cabin", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.blurred-background {
  background: url("/background/background.png");
  background-position: bottom;
  background-size: cover;
  background-repeat: no-repeat;
  filter: blur(2px);
  transform: scale(1.02);
}

.content {
  background: rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}

canvas {
  height: 170px;
  margin-top: 7rem;
}

.song-progress {
  height: 3px;
  background: rgba(255, 255, 255, 0.3);
  margin: 5px 0;
}

.thumbnail {
  width: 130px;
  height: 130px;
}

.thumbnail + div {
  letter-spacing: 1px;
}

.song-name {
  font-size: 2.4rem;
}
</style>
