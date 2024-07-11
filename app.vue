<template>
  <div id="app" class="flex flex-col items-center">
    
    <!-- Enter URL -->
    <input id="Enter-URL" v-model="videoUrl" @change="loadVideo" placeholder="Enter video URL"
           class="mb-4 p-2 text-black border border-gray-500 rounded w-full"/>

    <!-- Video Player -->
    <video
      ref="videoPlayer"
      id="videoPlayer"
      class="video-js vjs-default-skin rounded-lg shadow-lg"
      controls
      preload="auto"
    ></video>

    <!-- Control Bar -->
    <div id="ControlBar" class="flex mt-1 justify-between items-center">

      <div  style="display: inline-block;"> 
        <!-- Play | Pause -->
        <button @click="playPauseVideo"
                class="px-4 py-1 bg-white border border-gray-500 text-black text-lg rounded-md">
          {{ isPlaying ? '||' : "▷" }}
        </button>
      </div>

      <div  style="display: inline-block;"> 
        <!-- Backward | Forward -->
        <button @click="skipVideo(-skipSeconds)"
                class="px-4 py-1 bg-white border border-gray-500 text-black text-lg rounded-md">
                ↻
        </button>
        <input type="number" v-model.number="skipSeconds"  placeholder="Skip seconds"
              class="p-1 border border-gray-400 rounded w-20 text-center"/>
        <button @click="skipVideo(skipSeconds)"
                class="px-4 py-1 bg-white border border-gray-500 text-black text-lg rounded-md">
                ↺
        </button>
      </div>
      
        <!-- A-B Repeat controls -->
        <div  class="flex items-center"> 
        <label class="flex items-center">
          START
          <input type="number" v-model.number="repeatStart" :max="maxVideoSeconds"
                placeholder="Start timestamp"
                class="p-1 border border-gray-400 rounded w-20 text-center ml-4"/>
        </label>
        <label class="flex items-center ml-1">
          END
          <input type="number" v-model.number="repeatEnd" :max="maxVideoSeconds"
                placeholder="End timestamp"
                class="p-1 border border-gray-400  rounded w-20 text-center"/>
        </label>
        <button @click="toggleRepeatMode"
                class="px-4 py-1 bg-white border border-gray-500 text-black rounded-md ml-4">
          {{ isRepeating ? 'Stop A-B Repeat' : 'Start A-B Repeat' }}
        </button>
          </div>
    </div>

    <p v-if="isRepeating" class="mt-3 text-red-500">A-B REPEAT ON</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      videoUrl: '',
      skipSeconds: 10,
      repeatStart: 0,
      repeatEnd: 0,
      isPlaying: false,
      isRepeating: false,
      repeatInterval: null,
      maxVideoSeconds: 0,
    };
  },
  methods: {
    loadVideo() {
      const videoPlayer = this.$refs.videoPlayer;
      videoPlayer.src = this.videoUrl;
      videoPlayer.load();
      videoPlayer.addEventListener('loadedmetadata', () => {
        this.maxVideoSeconds = Math.floor(videoPlayer.duration);
      });
    },
    playPauseVideo() {
      const videoPlayer = this.$refs.videoPlayer;
      if (videoPlayer.paused) {
        videoPlayer.play();
        this.isPlaying = true;
      } else {
        videoPlayer.pause();
        this.isPlaying = false;
      }
    },
    skipVideo(seconds) {
      const videoPlayer = this.$refs.videoPlayer;
      videoPlayer.currentTime += seconds;
    },
    toggleRepeatMode() {
      // clear interval if user want to stop repeat
      if (this.isRepeating) {
        clearInterval(this.repeatInterval);
        this.isRepeating = false;
      } else {
        // verify if start and end are valid
        if (this.repeatStart >= 0 && this.repeatEnd > this.repeatStart) {
          this.isRepeating = true;
          this.repeatInterval = setInterval(() => {
            const videoPlayer = this.$refs.videoPlayer;
            // check if repeatEnd is reached
            if (videoPlayer.currentTime >= this.repeatEnd) {
              videoPlayer.currentTime = this.repeatStart;
            }
          }, 100);
        }
      }
    },
  },
  watch: {
    // Stop the repeat mode when video is paused
    isPlaying(newVal) {
      if (!newVal && this.isRepeating) {
        clearInterval(this.repeatInterval);
        this.isRepeating = false;
      }
    },
  },
};
</script>

<style scoped>
#app {
  text-align: center;
  margin-top: 50px;
}

#Enter-URL{
  width: 80vw;
}

#ControlBar {
  width: 80vw;
}

video {
  display: block;
  margin: 10px auto;
  width: 80vw;
  height: 60vh;
}
input, button {
  margin: 5px;
}
</style>
