<template lang="pug">
  .videoblock
    video(:width='width', :poster="poster", ref='videoItem').videoblock__video
      source(
        v-for='item in source'
        :src='item.src'
        :type="'video/'+item.type"
        )
      |Your browser does not support HTML5 video.
    .videoblock__navs(:class='[ fullScreen ? "full" : " "]' v-show="controls")
      button(
        type='button'
        :class="[playBtn ? 'videoblock__btn_play' : 'videoblock__btn_pause', 'videoblock__btn']"
        @click.prevent='playPause()'
      ) {{ playBtn ? 'play' : 'pause' }}
      span {{time.current}}/{{time.minutes}}:{{time.seconds}}
      span.videoblock__btn.videoblock__btn_range.progress
        input(
          type="range"
          min="0"
          :max="progress.fin"
          step="1"
          v-model="progress.selected"
          v-on:change="progressSet()"
        )
      button(
        type='button'
        :class='[ mute ? "muted" : " ", "videoblock__btn videoblock__btn_mute"]'
        @click.prevent='muteToggle()'
      ) sound
      span.videoblock__btn.videoblock__btn_range
        input(
          type="range"
          min="0"
          max="1"
          step="0.01"
          v-model="volume"
          v-on:change="volumeSet()"
        )
      button(
        type='button'
        class="videoblock__btn videoblock__btn_screen"
        @click.prevent='fullScreenToggle()'
      ) full screen
</template>

<script>
const timeSerialize = (val) => {
  if (val < 10) {
    return '0' + val
  }
  return val
}

export default {
  name: 'videoplayer',
  props: {
    source: {required: true, type: Array},
    width: {type: Number, default: 400},
    poster: {type: String},
    muted: {type: Boolean, default: false},
    autoplay: {type: Boolean, default: false},
    controls: {type: Boolean, default: true}
  },
  data () {
    return {
      video: null,
      playBtn: true,
      progress: {
        fin: 0,
        selected: 0
      },
      mute: this.muted,
      volume: 0,
      time: {
        minutes: '0',
        seconds: '00',
        current: '0:00'
      },
      fullScreen: false
    }
  },
  methods: {
    play () {
      this.video.play()
      this.playBtn = false
    },
    pause () {
      this.video.pause()
    },
    playPause () {
      this.playBtn = !this.playBtn
      if (this.playBtn) {
        this.video.pause()
      } else {
        this.video.play()
      }
    },
    progressSet () {
      this.video.currentTime = this.progress.selected
    },
    setMuted () {
      this.video.muted = true
    },
    volumeSet () {
      this.mute = false
      this.video.muted = false
      this.video.volume = this.volume
    },
    muteToggle () {
      if (this.mute) {
        this.video.muted = false
      } else {
        this.video.muted = true
      }
      this.mute = !this.mute
    },
    fullScreenToggle () {
      if (!this.fullScreen) {
        if (this.video.requestFullscreen) {
          this.video.requestFullscreen()
        } else if (this.video.msRequestFullscreen) {
          this.video.msRequestFullscreen()
        } else if (this.video.mozRequestFullScreen) {
          this.video.mozRequestFullScreen()
        } else if (this.video.webkitRequestFullscreen) {
          this.video.webkitRequestFullscreen()
        }
      } else {
        if (this.video.requestFullscreen) {
          this.video.exitFullscreen()
        } else if (this.video.msRequestFullscreen) {
          this.video.msExitFullscreen()
        } else if (this.video.mozRequestFullScreen) {
          this.video.mozExitFullScreen()
        } else if (this.video.webkitRequestFullscreen) {
          this.video.webkitExitFullscreen()
        }
      }
      this.fullScreen = !this.fullScreen
    },
    timeCheck () {
      let currentMinutes = parseInt(this.video.currentTime / 60, 10)
      let currentSeconds = Math.floor(this.video.currentTime % 60)
      return currentMinutes + ':' + timeSerialize(currentSeconds)
    }
  },
  mounted () {
    // init video element
    this.video = this.$refs.videoItem

    // check if all video data is loaded
    let i = setInterval(() => {
      if (this.video.readyState > 0) {
        this.time.minutes = parseInt(this.video.duration / 60, 10)
        this.time.seconds = Math.floor(this.video.duration % 60)
        this.progress.fin = this.video.duration
        clearInterval(i)
      }
    }, 200)

    this.video.addEventListener('timeupdate', () => {
      this.time.current = this.timeCheck()
      this.progress.selected = this.video.currentTime
    })
    // catch fullScreen changes

    if (document.addEventListener) {
      document.addEventListener('webkitfullscreenchange', exitHandler, false)
      document.addEventListener('mozfullscreenchange', exitHandler, false)
      document.addEventListener('fullscreenchange', exitHandler, false)
      document.addEventListener('MSFullscreenChange', exitHandler, false)
    }

    let self = this
    function exitHandler () {
      let videoStyles = window.getComputedStyle(self.video)
      if (document.webkitIsFullScreen || document.mozFullScreen || document.msFullscreenElement !== null) {
        if (videoStyles.zIndex > 9999) {
          return
        } else {
          self.fullScreen = false
        }
      }
    }

    // check options
    if (this.mute) {
      this.setMuted()
    }
    if (this.autoplay) {
      this.play()
    }
  }
}
</script>

<style lang="scss" scoped>
  *{
    box-sizing: border-box;
  }
  .videoblock{
    position: relative;
    video::-webkit-media-controls {
      display: none;
    }
    &__video{
      width: 100%;
    }
    &__navs{
      position: absolute;
      width: 100%;
      height: 25px;
      bottom: 0;
      left: 0;
      background: #ddd;
      &.full{
        position: absolute;
        z-index: 9999999999;
        bottom: 0;
        left: 0;
        width: 100%;
      }
    }
    &__btn{
      &.muted{
        text-decoration: line-through;
      }
      &_range{
        width: 100px;
        display: inline-block;
        input[type=range] {
        	/*removes default webkit styles*/
        	-webkit-appearance: none;

        	/*fix for FF unable to apply focus style bug */
        	border: 1px solid white;
          margin: 0;
        	/*required for proper track sizing in FF*/
        	width: 100%;
        }

        /* Webkit, Chrome & Safari */
        input[type=range]::-webkit-slider-runnable-track {
        	width: 300px;
        	height: 5px;
        	background: #ccc;
        	border: none;
        	border-radius: 3px;
        }
        input[type=range]::-webkit-slider-thumb {
        	-webkit-appearance: none;
        	border: none;
        	height: 16px;
        	width: 16px;
        	border-radius: 50%;
        	background: #004d66;
        	margin-top: -7px;
        }
        input[type=range]:focus {
        	outline: none;
        }
        input[type=range]:focus::-webkit-slider-runnable-track {
        	background: #ddd;
        }
        /* moz://a Firefox */
        input[type=range]::-moz-range-track {
        	/* width: 150px;
        	height: 5px; */
        	background: #ccc;
        	border: none;
        	border-radius: 3px;
        }
        input[type=range]::-moz-range-thumb {
        	border: none;
        	height: 16px;
        	width: 16px;
        	border-radius: 50%;
        	background: #004d66;
        }
        input[type=range]::-moz-range-progress {
        	background: #33ccff;
        	border-radius: 10px;
        	height: 5px;
        }

        /*hide the outline behind the border*/
        input[type=range]:-moz-focusring{
        	outline: 1px solid white;
        	outline-offset: -1px;
        }

        /* Microsoft */
        input[type=range]::-ms-track {

        	height: 2px;
        	/*remove bg colour from the track, we'll use ms-fill-lower and ms-fill-upper instead */
        	background: transparent;

        	/*leave room for the larger thumb to overflow with a transparent border */
        	border-color: transparent;
        	border-width: 6px 0;

        	/*remove default tick marks*/
        	color: transparent;
        }
        input[type=range]::-ms-thumb {
        	border: none;
        	height: 16px;
        	width: 16px;
        	border-radius: 50%;
        	background: #004d66;
        	margin-top: 1px;
        }
        input[type=range]::-ms-fill-lower {
        	background: #33ccff;
        	border-radius: 10px;
        	height: 5px;
        }
        input[type=range]::-ms-fill-upper {
        	background: #ccc;
        	border-radius: 10px;
        }
        input[type=range]:focus::-ms-fill-lower {
        	background: #44ddff;
        }
        input[type=range]:focus::-ms-fill-upper {
        	background: #ddd;
        }
      }
    }
  }
</style>
