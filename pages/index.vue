<template>
  <div
    class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12"
  >
    <div
      class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20 w-full max-w-4xl mx-auto"
    >
      <h1 class="text-3xl font-bold text-gray-900 mb-8 text-center">
        Interval Timer App
      </h1>
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
  selectedTimer.value = timer;
};
</script>
