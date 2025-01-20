<template>
  <div class="video-container" :class="theme">
    <div class="video-wrapper">
      <video ref="videoPlayer" class="video-js vjs-default-skin" controls>
        <source :src="videoSrc" type="application/dash+xml" />
      </video>
    </div>
    <div class="button-container">
      <button @click="playLive()" :disabled="isLive" class="control-button">
        Play Live
      </button>
      <button @click="playDash()" :disabled="isDash" class="control-button">
        Play DASH
      </button>
      <button @click="playHls()" :disabled="isHls" class="control-button">
        Play HLS
      </button>
    </div>
    <div class="button-container">
      <button
        @click="theme = ''"
        :disabled="theme === ''"
        class="control-button"
      >
        Default
      </button>
      <button
        @click="theme = 'forest-theme'"
        :disabled="theme === 'forest-theme'"
        class="control-button"
      >
        Forest
      </button>
    </div>
    <!-- <div class="button-container">
      <button @click="toggleMute()" class="control-button">
        {{ isMuted ? "Unmute" : "Mute" }}
      </button>
    </div> -->
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
const liveVideoSrc = ref(
  "https://ssh101stream.ssh101.com/akamaissh101/ssh101/stream53/chunks.m3u8"
);

const videoPlayer = ref(null);
const isDash = ref(true);
const isLive = ref(false);
const isHls = ref(false);
const theme = ref("");
const isMuted = ref(false);

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

const toggleMute = () => {
  if (!player) return;
  isMuted.value = !isMuted.value;
  player.muted(isMuted.value);
};

onMounted(() => {
  player = videojs(videoPlayer.value, {
    // playbackRates: [0.5, 1, 1.25, 1.5, 2],
    controls: true,
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

<style>
.video-js .vjs-playback-rate {
  color: white;
  border-radius: 5px;
  padding: 5px 10px;
  transition: background-color 0.3s ease;
}

.video-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #f4f4f4;
  padding: 20px;
  min-height: 100vh;
}

.video-wrapper {
  width: 100%;
  max-width: 1000px;
  margin-bottom: 20px;
  border: 4px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.video-js {
  width: 100%;
  height: 400px;
  border-radius: 8px;
}

.button-container {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-bottom: 15px;
}

.control-button {
  padding: 8px 14px;
  font-size: 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s ease-in-out;
}

.control-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.control-button:hover:not(:disabled) {
  background-color: #0056b3;
  transform: scale(1.05);
}

.control-button:active:not(:disabled) {
  background-color: #003f8f;
}

@media (max-width: 768px) {
  .video-container {
    padding: 10px;
  }

  .video-js {
    height: 200px;
  }

  .button-container {
    flex-direction: column;
    gap: 10px;
  }
}
</style>
