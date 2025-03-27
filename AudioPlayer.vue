<script setup>
import { ref, onMounted, computed, nextTick } from "vue";

let song = ref(null);
let isPlaying = ref(false);
let songDuration = ref(0);
let currentTime = ref(0);

function togglePlay() {
  if (!song.value) return;
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    song.value.play();
  } else {
    song.value.pause();
  }
}

onMounted(async () => {
  await nextTick();

  if (song.value) {
    song.value.pause();
  }
  song.value.addEventListener("loadedmetadata", () => {
    songDuration.value = song.value.duration;
  });
  song.value.addEventListener("timeupdate", () => {
    currentTime.value = song.value.currentTime;
  });
});

// ✅ Function to format seconds into MM:SS or HH:MM:SS
const formatTime = (seconds) => {
  let h = Math.floor(seconds / 3600);
  let m = Math.floor((seconds % 3600) / 60);
  let s = Math.floor(seconds % 60);

  let formattedTime =
    h > 0
      ? `${String(h).padStart(2, "0")}:${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`
      : `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;

  return formattedTime;
};

// ✅ Computed properties for formatted time
const formattedCurrentTime = computed(() => formatTime(currentTime.value));
const formattedDuration = computed(() => formatTime(songDuration.value));
</script>

<template>
  <div
    class="flex flex-col items-center bg-gray-900 text-white p-6 rounded-xl shadow-lg w-80"
  >
    <audio ref="song">
      <source src="../assets/her.mp3" type="audio/mp3" />
    </audio>

    <!-- Progress Bar -->
    <div class="w-full mb-4">
      <progress
        class="w-full h-2 bg-gray-700 rounded-full"
        :value="currentTime"
        :max="songDuration"
      ></progress>
      <div class="flex justify-between items-center">
        <p class="text-sm text-gray-400 mt-2">
          {{ formattedCurrentTime }}
        </p>
        <p class="text-sm text-gray-400 mt-2">
          {{ formattedDuration }}
        </p>
      </div>
    </div>

    <!-- Play Button -->
    <button
      @click="togglePlay"
      class="bg-red-500 hover:bg-red-600 text-white font-semibold py-2 px-6 rounded-full transition duration-300"
    >
      {{ isPlaying ? "⏸ Pause" : "▶ Play" }}
    </button>
  </div>
</template>
