<template>
  <form @submit.prevent="createTimer" class="space-y-6">
    <div>
      <label for="timerName" class="block text-sm font-medium text-gray-700"
        >Timer Name:</label
      >
      <input
        id="timerName"
        v-model="timerName"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label for="totalDuration" class="block text-sm font-medium text-gray-700"
        >Total Duration (minutes):</label
      >
      <input
        id="totalDuration"
        v-model="totalDuration"
        type="number"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label class="block text-sm font-medium text-gray-700 mb-2"
        >Intervals (seconds):</label
      >
      <div class="space-y-2">
        <div
          v-for="(interval, index) in intervals"
          :key="index"
          class="flex items-center"
        >
          <input
            v-model="intervals[index]"
            type="number"
            required
            class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
          />
          <button
            @click="removeInterval(index)"
            type="button"
            class="ml-2 inline-flex items-center p-1 border border-transparent rounded-full shadow-sm text-white bg-red-600 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500"
          >
            <svg
              class="h-5 w-5"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              aria-hidden="true"
            >
              <path
                fill-rule="evenodd"
                d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                clip-rule="evenodd"
              />
            </svg>
          </button>
        </div>
      </div>
      <button
        @click="addInterval"
        type="button"
        class="mt-2 inline-flex items-center px-3 py-2 border border-transparent text-sm leading-4 font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Add Interval
      </button>
    </div>
    <div>
      <label
        for="intervalThreshold"
        class="block text-sm font-medium text-gray-700"
        >Interval Warning Threshold (seconds):</label
      >
      <input
        id="intervalThreshold"
        v-model="intervalThreshold"
        type="number"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label for="intervalColor" class="block text-sm font-medium text-gray-700"
        >Interval Warning Color:</label
      >
      <input
        id="intervalColor"
        v-model="intervalColor"
        type="color"
        class="mt-1 block w-full h-10 rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label
        for="expireThreshold"
        class="block text-sm font-medium text-gray-700"
        >Timer Expiration Warning Threshold (seconds):</label
      >
      <input
        id="expireThreshold"
        v-model="expireThreshold"
        type="number"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label for="expireColor" class="block text-sm font-medium text-gray-700"
        >Timer Expiration Warning Color:</label
      >
      <input
        id="expireColor"
        v-model="expireColor"
        type="color"
        class="mt-1 block w-full h-10 rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <button
      type="submit"
      class="w-full py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
    >
      Create Timer
    </button>
  </form>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  initialTimer: {
    type: Object,
    default: null
  }
});

const timerName = ref("");
const totalDuration = ref(0);
const intervals = ref([60]);
const intervalThreshold = ref(10);
const expireThreshold = ref(10);
const intervalColor = ref("#FFFF00"); // Yellow
const expireColor = ref("#FF0000"); // Red

const emit = defineEmits(["create-timer"]);

watch(
  () => props.initialTimer,
  (newValue) => {
    if (newValue) {
      timerName.value = newValue.name;
      totalDuration.value = newValue.totalDuration;
      intervals.value = [...newValue.intervals];
      intervalThreshold.value = newValue.intervalThreshold || 10;
      expireThreshold.value = newValue.expireThreshold || 10;
      intervalColor.value = newValue.intervalColor || "#FFFF00";
      expireColor.value = newValue.expireColor || "#FF0000";
    }
  },
  { immediate: true }
);

const addInterval = () => {
  intervals.value.push(60);
};

const removeInterval = (index) => {
  intervals.value.splice(index, 1);
};

const createTimer = () => {
  emit("create-timer", {
    name: timerName.value,
    totalDuration: parseInt(totalDuration.value) * 60, // Convert to seconds
    intervals: intervals.value.map((interval) => parseInt(interval)),
    intervalThreshold: parseInt(intervalThreshold.value),
    expireThreshold: parseInt(expireThreshold.value),
    intervalColor: intervalColor.value,
    expireColor: expireColor.value
  });
  resetForm();
};

const resetForm = () => {
  timerName.value = "";
  totalDuration.value = 0;
  intervals.value = [60];
  intervalThreshold.value = 10;
  expireThreshold.value = 10;
  intervalColor.value = "#FFFF00";
  expireColor.value = "#FF0000";
};
</script>
