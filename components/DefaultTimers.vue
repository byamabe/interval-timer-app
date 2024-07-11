<template>
  <div class="mb-8">
    <h2 class="text-xl font-semibold text-gray-900 mb-4">Default Timers</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
      <button
        v-for="(timer, index) in defaultTimers"
        :key="index"
        @click="selectTimer(timer)"
        class="p-4 bg-white border border-gray-200 rounded-lg shadow-sm hover:bg-gray-50 transition-colors duration-150 text-left"
      >
        <h3 class="font-medium text-gray-900">{{ timer.name }}</h3>
        <p class="text-sm text-gray-500">
          Total: {{ formatDuration(timer.totalDuration) }}
        </p>
        <p class="text-sm text-gray-500">
          Intervals: {{ formatIntervals(timer.intervals) }}
        </p>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const defaultTimers = [
  {
    name: "3 min - 30 sec intervals",
    totalDuration: 3 * 60,
    intervals: Array(6).fill(30),
    intervalThreshold: 10,
    expireThreshold: 10
  },
  {
    name: "10 min - 30 sec intervals",
    totalDuration: 10 * 60,
    intervals: Array(20).fill(30),
    intervalThreshold: 10,
    expireThreshold: 10
  },
  {
    name: "10 min - alternating 60/30 sec",
    totalDuration: 10 * 60,
    intervals: Array(10)
      .fill(null)
      .flatMap(() => [60, 30]),
    intervalThreshold: 10,
    expireThreshold: 10
  }
];

const emit = defineEmits(["select-timer"]);

const selectTimer = (timer) => {
  emit("select-timer", timer);
};

const formatDuration = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  return `${minutes} min`;
};

const formatIntervals = (intervals) => {
  if (intervals.every((interval) => interval === intervals[0])) {
    return `${intervals[0]}s x ${intervals.length}`;
  } else {
    return intervals.join("s, ") + "s";
  }
};
</script>
