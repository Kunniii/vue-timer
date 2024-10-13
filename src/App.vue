<template>
  <div class="flex flex-col items-center justify-center h-screen bg-gray-100">
    <input
      type="text"
      v-model.trim="taskTitle"
      placeholder="Enter title for countdown"
      class="border px-2 py-1 rounded"
    />
    <div class="text-4xl font-bold my-4">{{ formattedTime }}</div>
    <div class="flex space-x-4">
      <div class="flex flex-col text-center">
        <label>Hours</label>
        <input
          type="number"
          v-model.number="targetHours"
          placeholder="0"
          class="w-20 border px-2 py-1 rounded"
        />
      </div>
      <div class="flex flex-col text-center">
        <label>Minutes</label>
        <input
          type="number"
          v-model.number="targetMinutes"
          placeholder="0"
          class="w-20 border px-2 py-1 rounded"
        />
      </div>
      <div class="flex flex-col text-center">
        <label>Seconds</label>
        <input
          type="number"
          v-model.number="targetSeconds"
          placeholder="0"
          class="w-20 border px-2 py-1 rounded"
        />
      </div>
    </div>
    <div class="flex gap-5">
      <button
        @click="resetCountdown"
        class="mt-2 bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded"
      >
        Reset
      </button>
      <button
        @click="startCountdown"
        class="mt-2 bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
      >
        Start
      </button>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted, onUnmounted, watch } from "vue";

  const taskTitle = ref("");
  const targetHours = ref(0);
  const targetMinutes = ref(0);
  const targetSeconds = ref(0);
  const remainingTime = ref(0);
  const formattedTime = ref("00:00:00");
  let intervalId = null;

  onMounted(() => {
    if ("Notification" in window) {
      Notification.requestPermission();
    }
  });

  const startCountdown = () => {
    clearInterval(intervalId);
    const targetTime =
      targetHours.value * 3600 + targetMinutes.value * 60 + targetSeconds.value;
    if (targetTime <= 0) return;
    remainingTime.value = targetTime;
    if (remainingTime.value > 0) {
      showNotification(`${taskTitle.value} started!`);
    }
    intervalId = setInterval(() => {
      remainingTime.value--;
      if (remainingTime.value < 0) {
        clearInterval(intervalId);
        remainingTime.value = 0;
      }
      formattedTime.value = formatTime(remainingTime.value);
    }, 1000);
  };

  const resetCountdown = () => {
    clearInterval(intervalId);
    targetHours.value = 0;
    targetMinutes.value = 0;
    targetSeconds.value = 0;
    remainingTime.value = 0;
    formattedTime.value = "00:00:00";
  };

  watch(remainingTime, (newTime) => {
    if (newTime === 0 && intervalId) {
      clearInterval(intervalId);
      showNotification(`${taskTitle.value} finished!`);
      intervalId = null;
    }
  });

  watch([targetHours, targetMinutes, targetSeconds], () => {
    const totalSeconds =
      targetHours.value * 3600 + targetMinutes.value * 60 + targetSeconds.value;
    formattedTime.value = formatTime(totalSeconds);
  });

  const formatTime = (seconds) => {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const remainingSeconds = seconds % 60;
    return `${String(hours).padStart(2, "0")}:${String(minutes).padStart(
      2,
      "0"
    )}:${String(remainingSeconds).padStart(2, "0")}`;
  };

  const showNotification = (message) => {
    if ("Notification" in window && Notification.permission === "granted") {
      new Notification(message);
    } else if ("Notification" in window) {
      Notification.requestPermission().then((permission) => {
        if (permission === "granted") {
          new Notification(message);
        }
      });
    } else {
      alert(message);
    }
  };

  onUnmounted(() => {
    clearInterval(intervalId);
  });
</script>
