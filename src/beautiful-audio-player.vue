<template>
  <div>
    <AudioCore :ref="audioCoreId"
               :id="audioCoreId"
               :src="src"
               @on-duration-change="handleDurationChange"
               @on-can-play="handleCanPlay"
               @on-progress="handleProgress"
               @on-play="handlePlay"
               @on-pause="handlePause"
               @on-ended="handleEnded"
               @on-time-update="handleTimeUpdate"
               @on-volume-change="handleVolumeChange">

      <div slot="controls"
           class="container">
        <div :class="{'bubble-content':true, 'not-read': readBadge} ">

          <div class="play-icon-container"
               :disabled="!canPlay"
               @click="handleToggle">
            <div :class="{'play-icon':true, pause: !playing, play:playing, disabled:!canPlay}"></div>
          </div>

          <div class="loader-container">
            <!-- <p class="loadingText">Loading</p> -->
          </div>

          <span class="audio-bar"
                :style="{width:trackWidth+'px'}">

            <span @mousemove="handleOverTrack"
                  @mouseout="handleOutTrack"
                  @click="handleClickTrack"
                  :class="{disabled: !canPlay, 'play-track-container':true}">
              <span :ref='playTrackId'
                    class="play-track"></span>
              <span class="played-track"
                    :style="{width: widthRate*100+'%'}"></span>
              <span v-if="showBuffer"
                    v-for="(item, i) in buffered"
                    class="buffered-track"
                    :key="i"
                    :style="{
                                left: 'calc(0.7rem +' +' '+(item.start/duration*100)+'%)',
                                width: (item.end-item.start)/duration*100+'%',
                            }"></span>
            </span>

            <div v-show="dragging||hovering"
                 :draggable="false"
                 class="float-container"
                 :style="{left:hovering?
                             'calc(0.6rem + '+widthRateHovering*100+'%)':
                             'calc(0.6rem + '+widthRate*100+'%)'
                             }">
              <div :draggable="false"
                   class="float-bubble">{{floatContainerTimeDisplay}}</div>
              <div :draggable="false"
                   class="triangle"></div>
              <div :draggable="false"
                   class="line-pointer"></div>
            </div>

            <span :ref='dragBlockId'
                  :draggable="false"
                  :style="{left: widthRate*100+'%'}"
                  class="drag-block"></span>
            <span v-if="showTime"
                  class="current-progress">
              <span class="current-time">{{currentTimeDisplay}}</span>
              <span class="total-time">{{durationDisplay}}</span>
            </span>
          </span>
          </span>
          <!-- <var></var> -->

        </div>
      </div>

    </AudioCore>
  </div>
</template>

<script>
import AudioCore from "./player-core.vue";

function getLayerXY(evt) {
  var el = evt.target,
    x = 0,
    y = 0;

  while (el && !isNaN(el.offsetLeft) && !isNaN(el.offsetTop)) {
    x += el.offsetLeft - el.scrollLeft;
    y += el.offsetTop - el.scrollTop;
    el = el.offsetParent;
  }

  x = evt.clientX - x;
  y = evt.clientY - y;

  return { x: x, y: y };
}

export default {
  name: "audioPlayer",
  props: {
    src: String,
    id: String,
    width: { type: Number, default: 200 },
    readBadge: Boolean,
    showTime: Boolean,
    showBuffer: Boolean,
  },
  components: { AudioCore },

  data() {
    return {
      audioRef: null,
      playing: false,
      canPlay: false,
      duration: 0,
      widthRate: 0,
      widthRateHovering: 0,
      dragBlockEle: null,
      dragging: false,
      hovering: false,
      buffered: [],
    };
  },
  computed: {
    trackWidth: function() {
      return this.width;
    },

    audioCoreId: function() {
      return this.id + "-audioCore";
    },
    dragBlockId: function() {
      return this.id + "-dragBlock";
    },
    playTrackId: function() {
      return this.id + "-playTrack";
    },
    durationDisplay: function() {
      return this.formatTime(this.duration);
    },
    currentTimeDisplay: function() {
      return this.formatTime(this.duration * this.widthRate);
    },
    currentTimeHoveringDisplay: function() {
      return this.formatTime(this.duration * this.widthRateHovering);
    },
    floatContainerTimeDisplay: function() {
      if (this.hovering) {
        return this.currentTimeHoveringDisplay;
      } else {
        return this.currentTimeDisplay;
      }
    },
    currentProgressDisplay: function() {
      return `${this.currentTimeDisplay}/${this.durationDisplay}`;
    },
  },

  mounted() {
    this.audioRef = this.$refs[this.audioCoreId];
    this.dragBlockEle = this.$refs[this.dragBlockId];
    this.playTrackEle = this.$refs[this.playTrackId];
    this.play = this.audioRef.play;
    this.pause = this.audioRef.pause;
    this.end = this.audioRef.end;
    this.restart = this.audioRef.restart;
    this.changeTime = this.audioRef.changeTime;
    this.changeVolume = this.audioRef.changeVolume;
    // window.addEventListener("mousemove", this.handleMouseMove);
    // window.addEventListener("mouseup", this.handleMouseUp);
    window.addEventListener("dragend", this.handleMouseUp);
  },

  methods: {
    handleToggle() {
      if (!this.canPlay) {
        console.log("not loaded, can not play now");
        return;
      }
      if (this.playing) {
        this.pause();
      } else {
        this.play();
      }
    },
    handleCanPlay() {
      this.canPlay = true;
      this.$emit("on-can-play", this.id);
    },
    handleProgress(buffered) {
      if (buffered.length > 0) {
        this.buffered = new Array(buffered.length).fill(1).map((v, i) => {
          const start = buffered.start(i);
          const end = buffered.end(i);
          console.log("progress", start, end);
          return { start, end };
        });
        this.$emit("on-progress", this.id);
      }
    },
    handlePlay() {
      this.playing = true;
      this.$emit("on-play", this.id);
    },
    handlePause() {
      this.playing = false;
      this.$emit("on-pause", this.id);
    },
    handleEnded(e) {
      this.playing = false;
      this.$emit("on-ended", this.id);
    },
    handleVolumeChange(volume) {
      console.log("volume change", volume);
      this.$emit("on-volume-change", this.id);
    },
    handleError(e) {
      this.$emit("on-error", this.id);
    },
    handleDurationChange(duration) {
      this.duration = duration;
    },
    handleTimeUpdate(rate) {
      this.widthRate = rate;
    },

    formatTime: function(seconds) {
      var seconds = Number(seconds);
      if (seconds > 60) {
        var minutes = Math.floor(seconds / 60);
        var remainSeconds = (seconds - minutes * 60).toFixed(0);
        return minutes + "' " + remainSeconds + '"';
      } else {
        return seconds.toFixed(0) + '"';
      }
    },
    // handleMouseDown(e) {
    //   const dragBlockEle = e.target;
    //   this.dragBlockEle = dragBlockEle;
    //   this.dragging = true;
    //   console.log("mousedown", e.target);
    // },
    // handleMouseMove(e) {
    //   if (this.dragging) {
    //     const movementX = e.movementX;
    //     const currentTimeRate =
    //       (movementX + this.widthRate * this.trackWidth) / this.trackWidth;
    //     console.log("move", e, movementX, this.widthRate, currentTimeRate);
    //     this.changeTime(currentTimeRate);
    //   }
    // },
    // handleMouseUp(e) {
    //   if (this.dragging) {
    //     console.log("up", e);
    //     this.dragBlockEle.removeEventListener(
    //       "mousemove",
    //       this.handleMouseMove
    //     );
    //     this.dragging = false;
    //     setTimeout(this.play, 0);
    //   }
    // },
    handleOverTrack(e) {
      if (this.canPlay) {
        this.hovering = true;
        const layerX = e.target.layerX || getLayerXY(e).x;
        const widthRateHovering = layerX / this.trackWidth;
        this.widthRateHovering = widthRateHovering;
      }
    },
    handleOutTrack(e) {
      this.hovering = false;
      this.widthRateHovering = 0;
    },
    handleClickTrack(e) {
      if (this.hovering) {
        this.changeTime(this.widthRateHovering);
        this.play();
      }
    },
  },
};
</script>

<style scoped>
body {
  padding: 0;
  margin: 0;
}

.bubble-content {
  /*width: 18rem;*/
  position: relative;
  float: left;
  max-width: 85%;
  min-width: 1rem;
  height: 1rem;
  padding: 0.8rem;
  margin: 0.5rem 4rem 0.5rem 0.2rem;
  background: #fff;
  border-radius: 3px;
  line-height: 2.4rem;
  box-shadow: 0 0 9px 3px rgba(0, 0, 0, 0.1);
}
.container .disabled {
  cursor: not-allowed;
}
.play-icon-container .disabled {
  position: absolute;
  width: 1.1rem;
  height: 0;
  margin: 0;
  top: 50%;
  transform: rotate(45deg);
  background-color: #ff6347;
  border-top: 1px solid #ff6347;
  border-bottom: 1px solid #ff6347;
  border-right: 1px solid #ff6347;
  border-left: 1px solid #ff6347;
}
.play-icon-container {
  display: inline-block;
  width: 1.2rem;
  height: 1.2rem;
  border-radius: 50%;
  font-size: 1rem;
  line-height: 2.5rem;
  color: #00acff;
  text-align: center;
  margin-left: 0.6rem;
  margin-top: 0.2rem;
  user-select: none;
  position: absolute;
  top: 0.5rem;
  box-shadow: 0 0 5px rgba(103, 132, 146, 0.6);
  cursor: pointer;
  border: 1px solid rgba(103, 132, 146, 0.4);
}

.play-icon-container[disabled] {
  cursor: not-allowed;
}

.loader-container {
  display: inline-block;
  display: none;
  width: 1rem;
  height: 1rem;
  line-height: 2.5rem;
  color: white;
  /*margin: 0 auto;*/
  margin-left: 1.2rem;
  margin-top: 0.8rem;
  position: absolute;
  top: 0.2rem;
  /*left: 50%;*/
  margin-right: -50%;
  transform: translate(-50%, -50%);
  border: 5px solid #3498db;
  border-radius: 50%;
  -webkit-animation: borderScale 1s infinite ease-in-out;
  animation: borderScale 1s infinite ease-in-out;
}

.not-read:after {
  content: "";
  display: block;
  width: 0.3rem;
  height: 0.3rem;
  border-radius: 50%;
  background: red;
  position: absolute;
  top: 0.2rem;
  right: -1rem;
}
.play-icon {
  position: relative;
  margin-left: 0.4rem;
  margin-top: 0.3rem;
}

.pause {
  /*content: "\e96a";*/
  width: 0;
  height: 0;
  border-top: 0.3rem solid transparent;
  border-left: 0.5rem solid #00a0e9;
  border-bottom: 0.3rem solid transparent;
  border-right: 0.5rem solid transparent;
}
.play {
  width: 0.3rem;
  height: 0.6rem;
  border-left: 0.15rem solid #00a0e9;
  border-right: 0.15rem solid #00a0e9;
  margin-left: 0.33rem;
  margin-top: 0.3rem;
}
.loading {
  background-image: url("../assets/loading.svg");
  width: 0.3rem;
  height: 0.6rem;
  margin-left: 0.33rem;
  margin-top: 0.3rem;
}
.drag-block {
  display: block;
  height: 0.9rem;
  width: 1.2rem;
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  background-color: #fff;
  box-shadow: 0 0 6px 3px rgba(197, 228, 243, 0.6);
  border-radius: 0.1rem;
  background-clip: padding-box;
  box-sizing: content-box;
  border-radius: 1.3rem;
  z-index: 10;
  transition: background-color ease 0.3s;
  background-color: #f8f8f8;
}
.drag-block::before {
  content: "";
  display: block;
  width: 1px;
  height: 0.3rem;
  background-color: #c5e4f3;
  margin: 0.3rem auto;
  top: 50%;
  z-index: 2;
}

.audio-bar {
  height: 100%;
  position: relative;
  display: inline-block;
  /*width: 100%;*/
  right: 3rem;
  margin-left: 5.5rem;
  vertical-align: top;
  margin-top: 1px;
}
.play-track-container {
  cursor: pointer;
}
.play-track {
  position: absolute;
  left: 0.7rem;
  right: 3.7rem;
  height: 0.5rem;
  width: 100%;
  background-color: rgba(165, 217, 243, 0.4);
  border-radius: 4px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 2;
}
.buffered-track {
  position: absolute;
  z-index: 1;
  background-color: springgreen;
  height: 0.5rem;
  top: 50%;
  transform: translateY(-50%);
  border-radius: 4px;
}
.played-track {
  position: absolute;
  display: block;
  left: 0.7rem;
  height: 0.5rem;
  background-image: linear-gradient(90deg, #00acff, #00eaff);
  border-radius: 4px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 4;
}

.float-container {
  width: 4rem;
  height: 3rem;
  background: transparent;
  position: absolute;
  left: 0.6rem;
  bottom: 0.85rem;
  transform: translateX(-50%);
  z-index: 2;
}
.float-bubble {
  position: relative;
  width: 90%;
  height: 1.5rem;
  user-select: none;
  border-radius: 1rem;
  background-color: yellow;
  margin: 0 auto;
  font-size: 1rem;
  text-align: center;
  line-height: 1.8rem;
  z-index: 1;
}
.triangle {
  width: 0;
  height: 0;
  border: 0.4rem solid transparent;
  border-top: 0.4rem solid yellow;
  border-bottom: 0.1 solid transparent;
  margin: 0 auto;
}
.line-pointer {
  width: 1px;
  height: 70%;
  background-color: yellow;
  margin: 0 auto;
}

.current-progress {
  font-size: 0.3rem;
  width: calc(100% + 30px);
  display: flex;
  justify-content: space-between;
  margin-top: 3px;
  margin-left: 3px;
}
</style>