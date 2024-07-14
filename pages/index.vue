<template>
  <div class="container mx-auto px-4 py-8">
    <h1 class="text-3xl font-bold text-gray-900 mb-8">Interval Timer App</h1>

    <DefaultTimers @select-timer="selectDefaultTimer" class="mb-8" />

    <div class="mb-8">
      <h2 class="text-2xl font-semibold text-gray-900 mb-4">
        Create or Edit Timer
      </h2>
      <TimerForm :initialTimer="selectedTimer" @create-timer="createTimer" />
    </div>

    <div v-if="activeTimer" class="mb-8">
      <h2 class="text-2xl font-semibold text-gray-900 mb-4">Active Timer</h2>
      <TimerDisplay :timer="activeTimer" />
    </div>

    <div v-if="savedTimers.length > 0" class="mb-8">
      <h2 class="text-2xl font-semibold text-gray-900 mb-4">Saved Timers</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <div
          v-for="(timer, index) in savedTimers"
          :key="index"
          class="bg-white shadow overflow-hidden sm:rounded-lg p-4"
        >
          <h3 class="text-lg font-medium text-gray-900">{{ timer.name }}</h3>
          <p class="mt-1 text-sm text-gray-600">
            Duration: {{ formatTime(timer.totalDuration) }}
          </p>
          <div class="mt-4 flex justify-between">
            <button
              @click="editTimer(timer)"
              class="inline-flex items-center px-3 py-2 border border-transparent text-sm leading-4 font-medium rounded-md text-indigo-700 bg-indigo-100 hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            >
              Edit
            </button>
            <button
              @click="activateTimer(timer)"
              class="inline-flex items-center px-3 py-2 border border-transparent text-sm leading-4 font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            >
              Activate
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import DefaultTimers from "../components/DefaultTimers.vue";
import TimerForm from "../components/TimerForm.vue";
import TimerDisplay from "../components/TimerDisplay.vue";

const selectedTimer = ref(null);
const activeTimer = ref(null);
const savedTimers = ref([]);

const selectDefaultTimer = (timer) => {
  selectedTimer.value = { ...timer };
};

const createTimer = (timerData) => {
  savedTimers.value.push(timerData);
  activateTimer(timerData);
  selectedTimer.value = null;
};

const editTimer = (timer) => {
  selectedTimer.value = { ...timer };
};

const activateTimer = (timer) => {
  activeTimer.value = { ...timer };
};

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return `${minutes}:${remainingSeconds.toString().padStart(2, "0")}`;
};
</script>
