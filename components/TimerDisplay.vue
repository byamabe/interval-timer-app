<template>
  <div
    class="p-6 my-4 rounded-lg shadow-md transition-colors duration-300"
    :style="{ backgroundColor: backgroundColor }"
  >
    <h2 class="text-2xl font-bold text-gray-900 mb-4">{{ timer.name }}</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-4">
      <div>
        <p class="text-sm font-medium text-gray-500">Time Remaining:</p>
        <p class="text-3xl font-bold text-gray-900">
          {{ formatTime(timeRemaining) }}
        </p>
      </div>
      <div>
        <p class="text-sm font-medium text-gray-500">Current Interval:</p>
        <p class="text-3xl font-bold text-gray-900">
          {{ currentInterval + 1 }} / {{ timer.intervals.length }}
        </p>
      </div>
    </div>
    <div class="mb-4">
      <p class="text-sm font-medium text-gray-500">Interval Time:</p>
      <p class="text-2xl font-bold text-gray-900">
        {{ formatTime(intervalTimeRemaining) }}
      </p>
    </div>
    <div
      v-if="currentIntervalData.title || currentIntervalData.description"
      class="mb-4 p-4 bg-white bg-opacity-50 rounded-md"
    >
      <h3
        v-if="currentIntervalData.title"
        class="text-lg font-semibold text-gray-900 mb-2"
      >
        {{ currentIntervalData.title }}
      </h3>
      <p v-if="currentIntervalData.description" class="text-gray-700">
        {{ currentIntervalData.description }}
      </p>
    </div>
    <div
      class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-2 mb-4"
    >
      <button
        @click="toggleTimer"
        class="flex-1 py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        {{ isRunning ? "Pause" : "Start" }}
      </button>
      <button
        @click="resetTimer"
        class="flex-1 py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Reset
      </button>
    </div>
    <div class="flex justify-between">
      <button
        @click="previousInterval"
        :disabled="currentInterval === 0"
        class="py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        Previous Interval
      </button>
      <button
        @click="nextInterval"
        :disabled="currentInterval === timer.intervals.length - 1"
        class="py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        Next Interval
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
const elapsedTime = ref(0);
const currentInterval = ref(0);

const timeRemaining = computed(() => {
  return Math.max(0, props.timer.totalDuration - elapsedTime.value);
});

const intervalTimeRemaining = computed(() => {
  const currentIntervalStartTime = props.timer.intervals
    .slice(0, currentInterval.value)
    .reduce((sum, interval) => sum + interval.duration, 0);

  return Math.max(
    0,
    props.timer.intervals[currentInterval.value].duration -
      (elapsedTime.value - currentIntervalStartTime)
  );
});

const backgroundColor = computed(() => {
  if (timeRemaining.value <= props.timer.expireThreshold) {
    return props.timer.expireColor;
  } else if (intervalTimeRemaining.value <= props.timer.intervalThreshold) {
    return props.timer.intervalColor;
  } else {
    return "white";
  }
});

const currentIntervalData = computed(() => {
  return props.timer.intervals[currentInterval.value];
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
    await playTone(440, 0.2);
    await new Promise((resolve) => setTimeout(resolve, 200));
  }
};

const playEndTone = async () => {
  for (let i = 0; i < 5; i++) {
    await playTone(587.33, 0.3);
    await new Promise((resolve) => setTimeout(resolve, 200));
  }
};

const toggleTimer = () => {
  if (isRunning.value) {
    clearInterval(intervalId);
  } else {
    intervalId = setInterval(tickTimer, 1000);
  }
  isRunning.value = !isRunning.value;
};

const tickTimer = async () => {
  if (elapsedTime.value < props.timer.totalDuration) {
    elapsedTime.value++;

    const currentIntervalEndTime = props.timer.intervals
      .slice(0, currentInterval.value + 1)
      .reduce((sum, interval) => sum + interval.duration, 0);

    if (elapsedTime.value >= currentIntervalEndTime) {
      if (currentInterval.value < props.timer.intervals.length - 1) {
        await playIntervalTone();
        currentInterval.value++;
      } else {
        await playEndTone();
        clearInterval(intervalId);
        isRunning.value = false;
      }
    }
  } else {
    await playEndTone();
    clearInterval(intervalId);
    isRunning.value = false;
  }
};

const resetTimer = () => {
  clearInterval(intervalId);
  isRunning.value = false;
  elapsedTime.value = 0;
  currentInterval.value = 0;
};

const nextInterval = async () => {
  if (currentInterval.value < props.timer.intervals.length - 1) {
    await playIntervalTone();
    elapsedTime.value = props.timer.intervals
      .slice(0, currentInterval.value + 1)
      .reduce((sum, interval) => sum + interval.duration, 0);
    currentInterval.value++;
  }
};

const previousInterval = () => {
  if (currentInterval.value > 0) {
    currentInterval.value--;
    elapsedTime.value = props.timer.intervals
      .slice(0, currentInterval.value)
      .reduce((sum, interval) => sum + interval.duration, 0);
  }
};

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = Math.floor(seconds % 60);
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
