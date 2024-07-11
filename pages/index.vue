<template>
  <div class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
    <h1 class="text-3xl font-bold text-gray-900 mb-8">Interval Timer App</h1>
    <DefaultTimers @select-timer="selectDefaultTimer" />
    <TimerForm
      @create-timer="addTimer"
      :initialTimer="selectedTimer"
      class="mb-8"
    />
    <div class="space-y-6">
      <TimerDisplay v-for="timer in timers" :key="timer.id" :timer="timer" />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import TimerForm from "~/components/TimerForm.vue";
import TimerDisplay from "~/components/TimerDisplay.vue";
import DefaultTimers from "~/components/DefaultTimers.vue";

const timers = ref([]);
const selectedTimer = ref(null);

const addTimer = (timerData) => {
  timers.value.push({
    id: Date.now(),
    ...timerData
  });
  selectedTimer.value = null;
};

const selectDefaultTimer = (timer) => {
  selectedTimer.value = {
    name: timer.name,
    totalDuration: timer.totalDuration / 60, // Convert to minutes for the form
    intervals: timer.intervals
  };
};
</script>
