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
        v-model.number="totalDuration"
        type="number"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>
    <div>
      <label class="block text-sm font-medium text-gray-700 mb-2"
        >Intervals:</label
      >
      <div class="space-y-4">
        <div
          v-for="(interval, index) in intervals"
          :key="index"
          class="p-4 bg-gray-50 rounded-md"
        >
          <div class="flex items-center justify-between mb-2">
            <span class="font-medium">Interval {{ index + 1 }}</span>
            <button
              @click="removeInterval(index)"
              type="button"
              class="text-red-600 hover:text-red-800"
            >
              Remove
            </button>
          </div>
          <div class="space-y-2">
            <input
              v-model.number="interval.duration"
              type="number"
              required
              placeholder="Duration (seconds)"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
            <input
              v-model="interval.title"
              placeholder="Title (optional)"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
            <textarea
              v-model="interval.description"
              placeholder="Description (optional)"
              rows="2"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            ></textarea>
          </div>
        </div>
      </div>
      <button
        @click.prevent="addInterval"
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
        v-model.number="intervalThreshold"
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
        v-model.number="expireThreshold"
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
const intervals = ref([{ duration: 60, title: "", description: "" }]);
const intervalThreshold = ref(10);
const expireThreshold = ref(10);
const intervalColor = ref("#FFFF00");
const expireColor = ref("#FF0000");

const emit = defineEmits(["create-timer"]);

watch(
  () => props.initialTimer,
  (newValue) => {
    if (newValue) {
      timerName.value = newValue.name;
      totalDuration.value = newValue.totalDuration / 60; // Convert to minutes
      intervals.value = newValue.intervals.map((interval) => ({
        duration: interval.duration,
        title: interval.title || "",
        description: interval.description || ""
      }));
      intervalThreshold.value = newValue.intervalThreshold || 10;
      expireThreshold.value = newValue.expireThreshold || 10;
      intervalColor.value = newValue.intervalColor || "#FFFF00";
      expireColor.value = newValue.expireColor || "#FF0000";
    }
  },
  { immediate: true }
);

const addInterval = () => {
  intervals.value.push({ duration: 60, title: "", description: "" });
};

const removeInterval = (index) => {
  intervals.value.splice(index, 1);
};

const createTimer = () => {
  emit("create-timer", {
    name: timerName.value,
    totalDuration: totalDuration.value * 60, // Convert to seconds
    intervals: intervals.value,
    intervalThreshold: intervalThreshold.value,
    expireThreshold: expireThreshold.value,
    intervalColor: intervalColor.value,
    expireColor: expireColor.value
  });
  resetForm();
};

const resetForm = () => {
  timerName.value = "";
  totalDuration.value = 0;
  intervals.value = [{ duration: 60, title: "", description: "" }];
  intervalThreshold.value = 10;
  expireThreshold.value = 10;
  intervalColor.value = "#FFFF00";
  expireColor.value = "#FF0000";
};
</script>
