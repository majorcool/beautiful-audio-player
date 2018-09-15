<template>
  <div class="container">
    <audio :id="id"
           :src="src"
           :controls="!hasControlSlot">
      <!-- <source src="" id="js-source" type="audio/mp3"/> -->
    </audio>
    <slot name="controls"></slot>
  </div>

</template>

<script>
export default {
  name: "audioCore",
  props: {
    src: String,
    id: String,
  },

  data() {
    return {
      audioEle: null,
      duration: 0,
      hasControlSlot: true,
    };
  },

  mounted() {
    const audio = document.getElementById(this.id);
    this.audioEle = audio;
    window.audioEle = this.audioEle;
    const hasControlSlot = this.$slots.controls;
    this.hasControlSlot = hasControlSlot;

    audio.addEventListener("play", this.onPlay);
    audio.addEventListener("pause", this.onPause);
    audio.addEventListener("ended", this.onEnded);
    audio.addEventListener("error", this.onError);
    audio.addEventListener("volumechange", this.onVolumeChange);
    audio.addEventListener("durationchange", this.onDurationChange);
    audio.addEventListener("timeupdate", this.onTimeupdate);
    audio.addEventListener("canplay", this.onCanPlay);
    audio.addEventListener("progress", this.onProgress);
  },
  computed: {},
  watch: {},

  methods: {
    onPlay(e) {
      this.$emit("on-play", e);
    },
    onPause(e) {
      this.$emit("on-pause", e);
    },
    onEnded(e) {
      this.$emit("on-ended", e);
    },
    onVolumeChange(e) {
      this.$emit("on-volume-change", this.audioEle.volume);
    },
    onError(e) {
      this.$emit("on-error", e);
    },
    onDurationChange() {
      this.duration = this.audioEle.duration;
      this.$emit("on-duration-change", this.duration);
    },
    onTimeupdate() {
      this.$emit("on-time-update", this.audioEle.currentTime / this.duration);
    },
    onCanPlay() {
      this.$emit("on-can-play");
    },
    onProgress(e) {
      this.$emit("on-progress", e.target.buffered);
    },

    play() {
      try {
        this.audioEle.play();
      } catch (e) {
        console.warn(e);
      }
    },
    changeTime(current, type = "rate") {
      if (type === "time") {
        if (current > this.duration || current < 0) {
          console.warn("audio-player.changeTime got an invalid parameter");
        } else {
          this.audioEle.currentTime = current;
        }
      } else if (type === "rate") {
        if (current > 1 || current < 0) {
          console.warn("audio-player.changeTime got an invalid parameter");
        } else {
          this.audioEle.currentTime = this.duration * current;
        }
      }
    },
    changeVolume(rate) {
      if (rate > 1 || rate < 0) {
        console.warn("audio-player.changeVolume got an invalid parameter");
      } else {
        this.audioEle.volume = rate;
        this.$emit("on-volume-change", rate);
      }
    },
    pause() {
      this.audioEle.pause();
      this.$emit("on-pause");
    },
    end() {
      this.audioEle.currentTime = 0;
      this.pause();
      this.$emit("on-ended");
    },
    restart() {
      this.audioEle.currentTime = 0;
      this.$emit("on-restart");
    },
  },
};
</script>
