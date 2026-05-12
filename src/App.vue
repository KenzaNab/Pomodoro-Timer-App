<template>
  <div class="app" :class="mode">
    <div class="container">
      <h1>🍅 Pomodoro Timer</h1>

      <div class="mode-tabs">
        <button v-for="m in modes" :key="m.key" :class="{ active: mode === m.key }" @click="switchMode(m.key)">
          {{ m.label }}
        </button>
      </div>

      <div class="timer-circle" :style="{ '--progress': progress }">
        <div class="timer-display">
          <span class="time">{{ formattedTime }}</span>
          <span class="mode-label">{{ currentMode.label }}</span>
        </div>
      </div>

      <div class="controls">
        <button @click="reset" class="btn-secondary">↺ Reset</button>
        <button @click="toggle" class="btn-primary">{{ running ? '⏸ Pause' : '▶ Start' }}</button>
        <button @click="skip" class="btn-secondary">⏭ Skip</button>
      </div>

      <div class="sessions">
        <span v-for="i in 4" :key="i" class="dot" :class="{ filled: i <= (completedPomodoros % 4) }" />
        <span class="sessions-count">{{ completedPomodoros }} sessions completed</span>
      </div>

      <div class="tasks-section">
        <div class="task-input">
          <input v-model="newTask" @keyup.enter="addTask" placeholder="Add a task..." />
          <button @click="addTask">+</button>
        </div>
        <div class="task-list">
          <div v-for="task in tasks" :key="task.id" class="task" :class="{ done: task.done }">
            <input type="checkbox" v-model="task.done" />
            <span>{{ task.text }}</span>
            <button @click="removeTask(task.id)" class="del">×</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

const MODES = {
  pomodoro: { label: 'Focus', duration: 25 * 60, key: 'pomodoro' },
  short: { label: 'Short Break', duration: 5 * 60, key: 'short' },
  long: { label: 'Long Break', duration: 15 * 60, key: 'long' },
}
const modes = [
  { key: 'pomodoro', label: 'Focus' },
  { key: 'short', label: 'Short Break' },
  { key: 'long', label: 'Long Break' },
]

const mode = ref('pomodoro')
const running = ref(false)
const timeLeft = ref(MODES.pomodoro.duration)
const completedPomodoros = ref(0)
const tasks = ref([])
const newTask = ref('')
let interval = null

const currentMode = computed(() => MODES[mode.value])
const progress = computed(() => timeLeft.value / currentMode.value.duration)
const formattedTime = computed(() => {
  const m = Math.floor(timeLeft.value / 60).toString().padStart(2, '0')
  const s = (timeLeft.value % 60).toString().padStart(2, '0')
  return `${m}:${s}`
})

function toggle() {
  running.value = !running.value
  if (running.value) {
    interval = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        clearInterval(interval)
        running.value = false
        if (mode.value === 'pomodoro') completedPomodoros.value++
        playBeep()
        auto_switch()
      }
    }, 1000)
  } else {
    clearInterval(interval)
  }
}

function switchMode(m) {
  clearInterval(interval); running.value = false
  mode.value = m; timeLeft.value = MODES[m].duration
}

function reset() {
  clearInterval(interval); running.value = false
  timeLeft.value = currentMode.value.duration
}

function skip() { switchMode(mode.value === 'pomodoro' ? 'short' : 'pomodoro') }

function auto_switch() {
  const next = mode.value === 'pomodoro'
    ? (completedPomodoros.value % 4 === 0 ? 'long' : 'short') : 'pomodoro'
  switchMode(next)
}

function playBeep() {
  const ctx = new AudioContext()
  const osc = ctx.createOscillator()
  osc.connect(ctx.destination)
  osc.frequency.value = 880
  osc.start(); osc.stop(ctx.currentTime + 0.3)
}

function addTask() {
  if (!newTask.value.trim()) return
  tasks.value.push({ id: Date.now(), text: newTask.value, done: false })
  newTask.value = ''
}

function removeTask(id) { tasks.value = tasks.value.filter(t => t.id !== id) }

onUnmounted(() => clearInterval(interval))
</script>

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
.app { min-height: 100vh; display: flex; align-items: center; justify-content: center; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; transition: background 0.5s; }
.app.pomodoro { background: #1a1a2e; color: #fff; }
.app.short { background: #0f3460; color: #fff; }
.app.long { background: #16213e; color: #fff; }
.container { width: 400px; text-align: center; padding: 2rem; }
h1 { font-size: 1.5rem; margin-bottom: 1.5rem; }
.mode-tabs { display: flex; gap: 8px; justify-content: center; margin-bottom: 2rem; }
.mode-tabs button { padding: 8px 16px; border: 1px solid rgba(255,255,255,0.3); border-radius: 8px; background: transparent; color: rgba(255,255,255,0.7); cursor: pointer; font-size: 14px; transition: all 0.2s; }
.mode-tabs button.active { background: rgba(255,255,255,0.2); color: #fff; border-color: rgba(255,255,255,0.5); }
.timer-circle { width: 240px; height: 240px; border-radius: 50%; margin: 0 auto 2rem; border: 8px solid rgba(255,255,255,0.1); display: flex; align-items: center; justify-content: center; background: conic-gradient(#e94560 calc(var(--progress) * 360deg), transparent 0deg); position: relative; }
.timer-circle::before { content: ''; position: absolute; width: 220px; height: 220px; border-radius: 50%; background: inherit; background: #1a1a2e; }
.timer-display { position: relative; z-index: 1; }
.time { display: block; font-size: 3.5rem; font-weight: 700; letter-spacing: 2px; }
.mode-label { font-size: 13px; opacity: 0.7; }
.app.short .timer-circle::before { background: #0f3460; }
.app.long .timer-circle::before { background: #16213e; }
.controls { display: flex; gap: 12px; justify-content: center; margin-bottom: 1.5rem; }
.btn-primary { padding: 12px 32px; background: #e94560; color: #fff; border: none; border-radius: 10px; font-size: 16px; font-weight: 600; cursor: pointer; }
.btn-secondary { padding: 12px 20px; background: rgba(255,255,255,0.1); color: #fff; border: none; border-radius: 10px; font-size: 14px; cursor: pointer; }
.sessions { display: flex; align-items: center; justify-content: center; gap: 8px; margin-bottom: 2rem; }
.dot { width: 12px; height: 12px; border-radius: 50%; background: rgba(255,255,255,0.2); }
.dot.filled { background: #e94560; }
.sessions-count { font-size: 13px; opacity: 0.6; margin-left: 4px; }
.tasks-section { background: rgba(255,255,255,0.05); border-radius: 16px; padding: 1rem; text-align: left; }
.task-input { display: flex; gap: 8px; margin-bottom: 12px; }
.task-input input { flex: 1; padding: 8px 12px; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); border-radius: 8px; color: #fff; font-size: 14px; outline: none; }
.task-input input::placeholder { color: rgba(255,255,255,0.4); }
.task-input button { padding: 8px 16px; background: #e94560; color: #fff; border: none; border-radius: 8px; cursor: pointer; font-size: 18px; }
.task { display: flex; align-items: center; gap: 8px; padding: 6px 0; font-size: 14px; }
.task.done span { text-decoration: line-through; opacity: 0.5; }
.task span { flex: 1; }
.del { background: none; border: none; color: rgba(255,255,255,0.3); cursor: pointer; font-size: 18px; padding: 0 4px; }
</style>
