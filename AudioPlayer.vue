<script setup>
import { ref, onMounted, computed, nextTick } from "vue";

const props = defineProps({
  audioPath: String,
  title: String,
});

let song = ref(null);
let isPlaying = ref(false);
let songDuration = ref(0);
let currentTime = ref(0);
let isSeeking = ref(false);

async function togglePlay() {
  if (!song.value) return;
  isPlaying.value = !isPlaying.value;
  try {
    if (isPlaying.value) {
      await song.value.play();
    } else {
      song.value.pause();
    }
  } catch (err) {
    console.error("Playback failed:", err);
    isPlaying.value = false;
  }
}

// ✅ Seek audio when dragging the slider
function updateTime(event) {
  if (!song.value) return;
  isSeeking.value = true;
  currentTime.value = event.target.value;
}

// ✅ Set audio time when slider is released
function seekAudio(event) {
  if (!song.value) return;
  song.value.currentTime = event.target.value;
  isSeeking.value = false;
}

onMounted(async () => {
  await nextTick();
  if (song.value) song.value.pause();

  song.value.addEventListener("loadedmetadata", () => {
    songDuration.value = song.value.duration || 0;
  });

  song.value.addEventListener("timeupdate", () => {
    if (!isSeeking.value) {
      currentTime.value = song.value.currentTime || 0;
    }
  });
});

// ✅ Format time as MM:SS or HH:MM:SS
const formatTime = (seconds) => {
  let h = Math.floor(seconds / 3600);
  let m = Math.floor((seconds % 3600) / 60);
  let s = Math.floor(seconds % 60);

  return h > 0
    ? `${String(h).padStart(2, "0")}:${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`
    : `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
};

// ✅ Computed time values (fallback to "00:00")
const formattedCurrentTime = computed(() =>
  isNaN(currentTime.value) ? "00:00" : formatTime(currentTime.value),
);
const formattedDuration = computed(() =>
  isNaN(songDuration.value) ? "00:00" : formatTime(songDuration.value),
);
</script>

<template>
  <div
    class="flex flex-col items-center bg-gray-800 text-white p-6 rounded-xl shadow-lg w-80"
  >
    <!-- ✅ Dynamic Audio Source -->
    <audio ref="song" :src="audioPath"></audio>

    <h1 class="text-xl font-bold font-mono italic m-2 text-center">
      {{ title }}
    </h1>

    <!-- Progress Bar (Clickable & Draggable) -->
    <div class="w-full mb-4">
      <input
        type="range"
        class="w-full h-1 bg-gray-600 rounded-lg appearance-none cursor-pointer accent-red-500"
        :value="currentTime"
        :max="songDuration"
        @input="updateTime"
        @change="seekAudio"
      />
      <div class="flex justify-between items-center text-xs text-gray-400 mt-1">
        <p>{{ formattedCurrentTime }}</p>
        <p>{{ formattedDuration }}</p>
      </div>
    </div>

    <!-- Play Button -->
    <button
      @click="togglePlay"
      class="bg-red-500 hover:bg-red-600 text-white font-semibold py-2 px-5 rounded-full transition duration-300 text-sm"
    >
      {{ isPlaying ? "⏸ Pause" : "▶ Play" }}
    </button>
  </div>
</template>
