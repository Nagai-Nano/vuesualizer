<template>
  <div id="app" class="w-screen h-screen overflow-hidden">
    <div class="w-full h-full flex items-center justify-center">
      <div class="self-end mb-32 w-3/5">
        <div class="text-center">
          <canvas ref="canvas" class="w-full h-32 border"></canvas>
          <div class="w-full mb-1" style="height: 3px; background: rgba(255, 255, 255, .3);">
            <div class="bg-white h-full" style="width: 10%;"></div>
          </div>
        </div>
        <div class="flex">
          <div class="mr-4">
            <img
              width="130"
              height="130"
              class="rounded-sm"
              src="/thumbnails/meant for you - OMFG.jpg"
            />
          </div>
          <div class="self-center mb-1 text-left">
            <h1 class="text-white tracking-wide font-bold uppercase mb-1">Meant For You</h1>
            <h3 class="text-grey-lighter tracking-wide font-hairline uppercase">OMFG</h3>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {}
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
