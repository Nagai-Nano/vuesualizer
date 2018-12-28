<template>
  <div id="app" class="w-screen h-screen overflow-hidden">
    <div class="w-full h-full flex items-center justify-center">
      <div class="self-end mb-32 w-3/5">
        <div class="text-center">
          <canvas ref="canvas" class="w-full h-32"></canvas>
          <div class="w-full mb-1" style="height: 3px; background: rgba(255, 255, 255, .3);">
            <div class="bg-white h-full" :style="{ width: progress }"></div>
          </div>
        </div>
        <div class="flex">
          <div class="mr-4">
            <img
              width="130"
              height="130"
              class="rounded-sm"
              :src="`/thumbnails/${thumbnails[index]}`"
              :alt="`/thumbnails/${thumbnails[index]}`"
            />
          </div>
          <div class="self-center mb-1 text-left">
            <h1 class="text-white tracking-wide font-bold uppercase mb-1">{{ songInfo.name }}</h1>
            <h3 class="text-grey-lighter tracking-wide font-hairline uppercase">{{ songInfo.artist }}</h3>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        songs: [],
        thumbnails: [],
        index: 0,
        audio: new Audio,
        progress: '0%'
      }
    },
    methods: {
      async fetchData() {
        const dataPaths = ['/songs', '/thumbnails']
        const promises = await Promise.all(dataPaths.map(path => axios.get(path)))

        this.songs = [...promises[0].data]
        this.thumbnails = [...promises[1].data]
      },

      init() {
        const canvas = this.$refs.canvas
        const ctx = canvas.getContext("2d")
        const { width, height } = canvas

        this.audio.src = `/songs/${this.songs[this.index]}`
        this.audio.play()

        const context = new AudioContext()
        const analyser = context.createAnalyser()
        const src = context.createMediaElementSource(this.audio)

        src.connect(analyser)
        analyser.connect(context.destination)
        // analyser.fftSize = 512
        // analyser.smoothingTimeConstant = 0.8

        renderFrame()
        function renderFrame() {
          requestAnimationFrame(renderFrame)

          const dataArray = new Uint8Array(analyser.frequencyBinCount)
          analyser.getByteFrequencyData(dataArray)

          ctx.clearRect(0, 0, width, height)
          ctx.fillStyle = '#fff'

          for (let i = 0, len = dataArray.length; i < 100; i++) {
            let barX = i * 7
            let barWidth = 6
            let barHeight = -(dataArray[i] / 2)

            ctx.fillRect(barX, height, barWidth, barHeight)
          }
        }
      },

      timeUpdate() {
        const newTime = this.audio.currentTime * (100 / this.audio.duration)
        this.progress = newTime + '%'
      },

      nextSong() {
        this.index++
        const next = this.songs[this.index]

        if(next) {
          this.audio.src = `/songs/${next}`
          this.audio.play()
        }
        else console.log('end')
      }
    },
    computed: {
      songInfo() {
        if(this.songs[this.index]) {
          const re = /\.(mp3|aac|ogg|m4a|wma|flac|wav)$/i
          const [ name, artist ] = this.songs[this.index].replace(re, '').split('-')
          return { name, artist }
        }

        return {
          name: 'Dunno',
          artist: 'Dunno'
        }
      }
    },
    async created() {
      await this.fetchData()

      this.init()
      this.audio.addEventListener('timeupdate', this.timeUpdate)
      this.audio.addEventListener('ended', this.nextSong)
    }
  }
</script>

<style>
  #app {
    font-family: 'Cabin', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background: url(/backgrounds/background2.jpg);
    background-size: cover !important;
    background-position: center !important;
  }

  #app > div {
    background: rgba(0, 0, 0, .3);
  }
</style>
