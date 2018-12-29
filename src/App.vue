<template>
  <div
    id="app"
    class="w-screen h-screen overflow-hidden relative"
    :style="{ background: `url(/background/${background})` }"
  >
    <transition
      name="custom-classes-transition"
      enter-active-class="animated bounceInLeft"
      leave-active-class="animated bounceOutRight"
    >
      <div
        v-if="upNext"
        id="upNext"
        class="w-full h-full absolute pin-t pin-l"
        :style="{ background: `url(/background/${background})` }"
      >
        <div
          class="w-full h-full flex flex-col items-center justify-center"
          style="background: rgba(0, 0, 0, .3);"
        >
          <h1
            class="text-black bg-white px-4 py-3 uppercase rounded shadow-md text-4xl text-center font-bold tracking-wide mt-8"
            style="opacity: .9; min-width: 25%; max-width: 65%;"
          >
            <i class="fa fa-forward"></i>
            {{ upNextSong.name }}
          </h1>
          <span
            class="text-white bg-black px-4 py-3 tracking-wide uppercase shadow-md text-center rounded"
            style="opacity: .9; min-width: 15%; max-width: 50%;"
          >
            {{ upNextSong.artist }}
          </span>
        </div>
      </div>
    </transition>
    <div
      class="w-full h-full flex items-center justify-center"
      style="background: rgba(0, 0, 0, .3);"
    >
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
              class="rounded-sm shadow"
              :src="`/thumbnails/${thumbnails[index]}`"
            />
          </div>
          <div class="self-center mb-1 text-left">
            <h1 class="text-white tracking-wide font-bold uppercase mb-1">{{ getSongInfo.name }}</h1>
            <h3 class="text-grey-lighter tracking-wide font-hairline uppercase">{{ getSongInfo.artist }}</h3>
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
        background: '',
        index: 4,
        audio: new Audio,
        progress: '0%',
        upNext: false,
        upNextSong: {}
      }
    },
    methods: {
      async fetchData() {
        const dataPaths = ['/songs', '/thumbnails', '/background']
        const promises  = await Promise.all(dataPaths.map(path => axios.get(path)))

        this.songs      = [...promises[0].data]
        this.thumbnails = [...promises[1].data]
        this.background = promises[2].data
      },

      songInfo(song) {
        const re = /\.(mp3|aac|ogg|m4a|wma|flac|wav)$/i
        const [ name, artist ] = song.replace(re, '').split('-')

        return { name, artist }
      },

      initPlayer() {
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
            let barHeight = -(dataArray[i] / 2.5)

            ctx.fillRect(barX, height, barWidth, barHeight)
          }
        }
      },

      timeUpdate() {
        const newTime = this.audio.currentTime * (100 / this.audio.duration)
        this.progress = newTime + '%'

        if(Math.floor(this.audio.duration - this.audio.currentTime) === 5) {
          if(this.songs[this.index + 1]) {
            this.upNext = true
            this.upNextSong = this.songInfo(this.songs[this.index + 1])
          }
        }
      },

      nextSong() {
        if(this.songs[this.index + 1]) {
          this.index++
          this.audio.src = `/songs/${this.songs[this.index]}`
          this.audio.play()

          setTimeout(() => {
            this.upNext = false
            this.preFetch()
          }, 500)
        }
        else console.log('tắt nghỉ :))')
      },

      preFetch() {
        const nextIndex = this.index + 1
        const nextDataPaths = [
          `/thumbnails/${this.thumbnails[nextIndex]}`,
          `/songs/${this.songs[nextIndex]}`,
        ]

        Promise.all(nextDataPaths.map(path => axios.get(path)))
      }
    },
    computed: {
      getSongInfo() {
        if(this.songs[this.index])
          return this.songInfo(this.songs[this.index])

        return {
          name: 'Dunno',
          artist: 'Dunno'
        }
      }
    },
    async created() {
      await this.fetchData()

      this.initPlayer()
      this.audio.addEventListener('timeupdate', this.timeUpdate)
      this.audio.addEventListener('ended', this.nextSong)

      this.preFetch()
    }
  }
</script>

<style>
  #app {
    font-family: 'Cabin', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  #app,
  #upNext {
    background-size: cover !important;
    background-position: center !important;
  }
</style>
