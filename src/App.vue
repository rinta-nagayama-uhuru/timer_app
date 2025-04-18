<template>
  <div id="app">
    <div>
      <label for="timeInput">カウントダウン時間(分):</label>
      <input id="timeInput" type="number" v-model.number="inputMinutes" step="1" />
    </div>
    <div class="timer">{{ formatTime }}</div>
    <ButtonControl
    :isSetDisabled="timerId !== null"
    :isStartDisabled="timerId !== null || !timeSet"
    :isStopDisabled="timerId === null"
    @setTime="setTime"
    @startTimer="startTimer"
    @stopTimer="stopTimer"
    @resetTimer="resetTimer"
    />
  </div>
</template>


<script setup>
  import { ref, computed, onMounted } from 'vue';
  import ButtonControl from './components/ButtonControl.vue';

  const inputMinutes = ref(0);
  const startingTime = ref(0);
  const time = ref(0);
  let timerId = ref(null);
  const timeSet = ref(false);
  
  function setTime() {
    startingTime.value = inputMinutes.value * 60; 
    time.value = startingTime.value;
    timeSet.value = true;
  }

  function startTimer() {
    localStorage.setItem('timerRunning', 'true');
    if (timerId.value === null && timeSet.value) {
      timerId.value = setInterval(() => {
        if (time.value > 0) {
          time.value -= 1;
          localStorage.setItem('savedTime', time.value);
          localStorage.setItem('savedTimestamp', Date.now());
          if (time.value === 0) {
            setTimeout(() => {
              alert("時間になりました!");
            }, 10);
            resetTimer();
          }
        } else {
          resetTimer();
        }
      }, 1000);
    }
  }

  function stopTimer() {
    clearInterval(timerId.value);
    timerId.value = null;
    localStorage.setItem('timerRunning', 'false');
  }

  function resetTimer() {
    clearInterval(timerId.value);
    timerId.value = null;
  time.value = startingTime.value;
  localStorage.removeItem('savedTime');
  localStorage.removeItem('savedTimestamp');
  localStorage.removeItem('timerRunning');
}
const formatTime = computed(() => {
  const minutes = Math.floor(time.value / 60);
  const seconds = ((time.value / 60) % 1) * 60;
  return `${minutes}.${seconds.toFixed(0).padStart(2, '0')}`;
});

onMounted(() => {
    const savedTime = localStorage.getItem('savedTime');
    const savedTimestamp = localStorage.getItem('savedTimestamp');
    const timerRunning = localStorage.getItem('timerRunning') === 'true';

  if (savedTime !== null && savedTimestamp !== null) {
      const currentTime = Date.now();
      const elapsedSeconds = Math.floor((currentTime - Number(savedTimestamp)) / 1000);
      let calculatedTime = Number(savedTime) - elapsedSeconds;
      
    if (calculatedTime > 0 && timerRunning) {
        time.value = calculatedTime;
        timeSet.value = true;
        startTimer();
      } else if (!timerRunning) {
        calculatedTime = Number(savedTime); 
        time.value = calculatedTime > 0 ? calculatedTime : 0;
        timeSet.value = true;
      } else {
        time.value = 0;
        startingTime.value = 0;
        timeSet.value = false;
      }
    }
  });
</script>

<style>
html, body {
  margin: 0;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
}

.timer {
  font-size: 2.5em;
  margin: 20px 0;
  padding: 10px 20px;
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  color: #4caf50;
}

div > div {
  margin-bottom: 15px;
}

label {
  margin-right: 10px;
}

input[type="number"] {
  font-size: 1em;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 3px;
  width: 80px; 
  box-sizing: border-box;
}

.controls {
  display: flex;
  gap: 10px;
}

button {
  cursor: pointer;
  font-size: 1em;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #2196f3;
  color: white;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #1976d2;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>