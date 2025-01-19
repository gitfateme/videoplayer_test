<template>
  <div class="video-container">
    <video
      ref="videoPlayer"
      class="video-js vjs-default-skin vjs-theme-sea"
      controls
    >
      <source :src="videoSrc" type="application/dash+xml" />
    </video>
    <div>
      <button @click="playLive()" :disabled="isLive">Play Live</button>
      <button @click="playDash()" :disabled="isDash">Play DASH</button>
      <button @click="playHls()" :disabled="isHls">Play HLS</button>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onBeforeUnmount } from "vue";
import videojs from "video.js";
import "video.js/dist/video-js.css";

const videoSrc = ref(
  "https://dash.akamaized.net/dash264/TestCasesIOP33/adapatationSetSwitching/5/manifest.mpd"
);
const hlsVideoSrc = ref(
  "https://d2zihajmogu5jn.cloudfront.net/sintel/master.m3u8"
);
//example live source
const liveVideoSrc = ref(
  "https://ssh101stream.ssh101.com/akamaissh101/ssh101/stream53/chunks.m3u8"
);

// Create a reference to the video element
const videoPlayer = ref(null);
const isDash = ref(true);
const isLive = ref(false);
const isHls = ref(false);

let player = null;

const playHls = () => {
  if (!player) return;
  isLive.value = false;
  isDash.value = false;
  isHls.value = true;
  player.src({
    src: hlsVideoSrc.value,
    type: "application/x-mpegURL",
  });
  player.play();
};

const playLive = () => {
  if (!player) return;
  isLive.value = true;
  isDash.value = false;
  isHls.value = false;

  player.src({
    src: liveVideoSrc.value,
    type: "application/x-mpegURL",
  });
  player.play();
};
const playDash = () => {
  if (!player) return;
  isLive.value = false;
  isDash.value = true;
  isHls.value = false;

  player.src({
    src: videoSrc.value,
    type: "application/dash+xml",
  });
  player.play();
};

onMounted(() => {
  player = videojs(videoPlayer.value, {
    // liveui: true, // Enable live UI
    // controlBar: {
    //   playToggle: true, // Show or hide play/pause button
    //   volumePanel: true, // Show or hide volume controls
    //   fullscreenToggle: true, // Show or hide fullscreen button
    //   progressControl: false, // Remove seek bar
    //   remainingTimeDisplay: false, // Remove remaining time display
    //   currentTimeDisplay: false, // Remove current time display
    // },
    // controls: true,
    // autoplay: false,
    // preload: "auto",
    // techOrder: ["dash"], // Ensure DASH is set as the tech

    qualitySelector: true,

    html5: {
      vhs: {},
    },
  });

  player.ready(() => {
    console.log("Video.js player is ready");
  });

  player.on("loadedmetadata", function () {
    var liveTracker = player.liveTracker;

    if (liveTracker) {
      if (liveTracker.isLive()) {
        console.log("The stream is live!");
        liveTracker.seekToLiveEdge();
      } else {
        console.log("The stream is not live.");
      }
    }
  });

  player.on("play", function () {
    var liveTracker = player.liveTracker;
    if (liveTracker) {
      if (liveTracker.isLive()) {
        console.log("The stream is live!");
        liveTracker.seekToLiveEdge();
      } else {
        console.log("The stream is not live.");
      }
    }
  });
});

onBeforeUnmount(() => {
  if (player) {
    player.dispose();
  }
});
</script>

<style scoped>
.video-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 40px;
}
.video-js {
  width: 900px;
  height: 300px;
}
</style>
