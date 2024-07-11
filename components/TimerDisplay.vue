<template>
  <div
    class="p-4 my-4 rounded-lg transition-colors duration-300"
    :style="{ backgroundColor: backgroundColor }"
  >
    <h2 class="text-2xl font-bold text-gray-900">{{ timer.name }}</h2>
    <p class="mt-2 text-lg text-gray-600">
      Time Remaining: {{ formatTime(timeRemaining) }}
    </p>
    <p class="mt-1 text-sm text-gray-500">
      Current Interval: {{ currentInterval + 1 }} / {{ timer.intervals.length }}
    </p>
    <p class="mt-1 text-sm text-gray-500">
      Interval Time: {{ formatTime(intervalTimeRemaining) }}
    </p>
    <div class="mt-4 space-x-2">
      <button
        @click="toggleTimer"
        class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        {{ isRunning ? "Pause" : "Start" }}
      </button>
      <button
        @click="resetTimer"
        class="inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md shadow-sm text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Reset
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";

const props = defineProps({
  timer: {
    type: Object,
    required: true
  }
});

const isRunning = ref(false);
const timeRemaining = ref(props.timer.totalDuration);
const currentInterval = ref(0);
const intervalTimeRemaining = ref(props.timer.intervals[0]);

const backgroundColor = computed(() => {
  if (timeRemaining.value <= props.timer.expireThreshold) {
    return props.timer.expireColor;
  } else if (intervalTimeRemaining.value <= props.timer.intervalThreshold) {
    return props.timer.intervalColor;
  } else {
    return "white";
  }
});

let intervalId = null;
let audioContext = null;

onMounted(() => {
  audioContext = new (window.AudioContext || window.webkitAudioContext)();
});

const playTone = (frequency, duration) => {
  return new Promise((resolve) => {
    const oscillator = audioContext.createOscillator();
    const gainNode = audioContext.createGain();

    oscillator.type = "sine";
    oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);

    gainNode.gain.setValueAtTime(1, audioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(
      0.001,
      audioContext.currentTime + duration
    );

    oscillator.connect(gainNode);
    gainNode.connect(audioContext.destination);

    oscillator.start();
    oscillator.stop(audioContext.currentTime + duration);

    setTimeout(resolve, duration * 1000);
  });
};

const playIntervalTone = async () => {
  for (let i = 0; i < 3; i++) {
    await playTone(440, 0.2); // 440 Hz - A4 note, for 0.2 seconds
    await new Promise((resolve) => setTimeout(resolve, 200)); // 200ms pause between tones
  }
};

const playEndTone = async () => {
  for (let i = 0; i < 5; i++) {
    await playTone(587.33, 0.3); // 587.33 Hz - D5 note, for 0.3 seconds
    await new Promise((resolve) => setTimeout(resolve, 200)); // 200ms pause between tones
  }
};

const toggleTimer = () => {
  if (isRunning.value) {
    clearInterval(intervalId);
  } else {
    intervalId = setInterval(async () => {
      if (timeRemaining.value > 0) {
        timeRemaining.value--;
        intervalTimeRemaining.value--;
        if (intervalTimeRemaining.value === 0) {
          if (currentInterval.value < props.timer.intervals.length - 1) {
            await playIntervalTone(); // Play interval tone 3 times
            currentInterval.value++;
            intervalTimeRemaining.value =
              props.timer.intervals[currentInterval.value];
          } else {
            await playEndTone(); // Play end tone 5 times
            clearInterval(intervalId);
            isRunning.value = false;
          }
        }
      } else {
        await playEndTone(); // Play end tone 5 times
        clearInterval(intervalId);
        isRunning.value = false;
      }
    }, 1000);
  }
  isRunning.value = !isRunning.value;
};

const resetTimer = () => {
  clearInterval(intervalId);
  isRunning.value = false;
  timeRemaining.value = props.timer.totalDuration;
  currentInterval.value = 0;
  intervalTimeRemaining.value = props.timer.intervals[0];
};

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return `${minutes}:${remainingSeconds.toString().padStart(2, "0")}`;
};

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }
  if (audioContext) {
    audioContext.close();
  }
});
</script>
