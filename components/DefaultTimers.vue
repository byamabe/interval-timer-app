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
    name: "3 min - 30 sec intervals (Neck Exercises)",
    totalDuration: 3 * 60,
    intervals: [
      {
        minutes: 0,
        seconds: 30,
        title: "Left to Right",
        description: "Look from left to right"
      },
      {
        minutes: 0,
        seconds: 30,
        title: "Up and Down",
        description: "Look up then down with hands on chest"
      },
      {
        minutes: 0,
        seconds: 30,
        title: "Tilt ears up from side to side",
        description:
          "Tilt ears up from side to side with hands on chest. Don't bring ears down."
      },
      {
        minutes: 0,
        seconds: 30,
        title: "Neck forward and back",
        description: "Move neck forward then back with hands on chest"
      },
      {
        minutes: 0,
        seconds: 30,
        title: "Eyes level move head side to side",
        description: "Move head side to side with eyes level like Janet Jackson"
      },
      {
        minutes: 0,
        seconds: 30,
        title: "Head half circles",
        description: "Tilt ear up then roll chin to chest"
      }
    ],
    intervalThreshold: 10,
    expireThreshold: 10,
    intervalColor: "#FFFF00",
    expireColor: "#FF0000"
  },
  {
    name: "10 min - 30 sec intervals",
    totalDuration: 10 * 60,
    intervals: Array(20)
      .fill()
      .map(() => ({ minutes: 0, seconds: 30, title: "", description: "" })),
    intervalThreshold: 10,
    expireThreshold: 10,
    intervalColor: "#FFFF00",
    expireColor: "#FF0000"
  },
  {
    name: "10 min - alternating 60/30 sec",
    totalDuration: 10 * 60,
    intervals: Array(10)
      .fill()
      .flatMap(() => [
        { minutes: 1, seconds: 0, title: "", description: "" },
        { duration: 30, title: "", description: "" }
      ]),
    intervalThreshold: 10,
    expireThreshold: 10,
    intervalColor: "#FFFF00",
    expireColor: "#FF0000"
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
  if (
    intervals.every((interval) => interval.duration === intervals[0].duration)
  ) {
    return `${intervals[0].duration}s x ${intervals.length}`;
  } else {
    return intervals.map((interval) => interval.duration).join("s, ") + "s";
  }
};
</script>
