<script setup>
import { ref, computed, reactive, watch, onMounted } from "vue"

// --- STATE NAVIGASI HALAMAN ---
const activeTab = ref("tasks") // Options: 'tasks' | 'analytics' | 'settings'

// --- STATE THEME / SETTINGS ---
const defaultTheme = {
  bgGradient: "from-slate-900 via-purple-900 to-slate-900",
  glassOpacity: "bg-white/35",
  accentColor: "text-cyan-500",
  orbColors: ["bg-pink-300/60", "bg-sky-300/60", "bg-blue-200/40"]
}

const theme = reactive({ ...defaultTheme })

// Load settings dari localStorage saat mount
onMounted(() => {
  const saved = localStorage.getItem("taskify-theme")
  if (saved) {
    try { Object.assign(theme, JSON.parse(saved)) } catch (e) {}
  }
})

// Auto-save setiap kali theme berubah
watch(theme, (val) => {
  localStorage.setItem("taskify-theme", JSON.stringify(val))
}, { deep: true })

const resetTheme = () => {
  if (confirm("Kembalikan ke pengaturan default?")) {
    Object.assign(theme, defaultTheme)
  }
}

// Opsi tema yang tersedia
const bgOptions = [
  { label: "Midnight Purple", value: "from-slate-900 via-purple-900 to-slate-900" },
  { label: "Ocean Deep", value: "from-slate-900 via-cyan-900 to-slate-900" },
  { label: "Sunset Warm", value: "from-slate-900 via-orange-900 to-slate-900" },
  { label: "Forest Calm", value: "from-slate-900 via-emerald-900 to-slate-900" },
  { label: "Rose Night", value: "from-slate-900 via-rose-900 to-slate-900" },
]

const glassOptions = [
  { label: "Light (35%)", value: "bg-white/35" },
  { label: "Medium (50%)", value: "bg-white/50" },
  { label: "Heavy (70%)", value: "bg-white/70" },
  { label: "Frost (20%)", value: "bg-white/20" },
]

const accentOptions = [
  { label: "Cyan", value: "text-cyan-500" },
  { label: "Pink", value: "text-pink-500" },
  { label: "Amber", value: "text-amber-500" },
  { label: "Emerald", value: "text-emerald-500" },
  { label: "Violet", value: "text-violet-500" },
]

// --- STATE TASK LOGIC ---
const tasks = ref([
  { id: 1, title: "Contoh 1", isDone: true },
  { id: 2, title: "Contoh 2", isDone: false }
])
const taskInput = ref("")

// Function Tambah Task
const addTask = () => {
  if (taskInput.value.trim() === "") {
    return alert("Task tidak boleh kosong")
  }
  const newTask = {
    id: Date.now(),
    title: taskInput.value.trim(),
    isDone: false
  }
  tasks.value.unshift(newTask)
  taskInput.value = ""
}

// Function Hapus Single Task
const removeTask = (id) => {
  const index = tasks.value.findIndex(task => task.id === id)
  if (index === -1) return alert(`Task dengan id ${id} tidak ditemukan!`)
  if (confirm(`Apakah Anda yakin ingin menghapus task ini?`)) {
    tasks.value.splice(index, 1)
  }
}

// Function Toggle Task
const toggleTask = (task) => {
  task.isDone = !task.isDone
}

// Function Hapus Semua Task
const clearAllTasks = () => {
  if (tasks.value.length === 0) return
  if (confirm("Apakah Anda yakin ingin menghapus SELURUH task?")) {
    tasks.value = []
  }
}

// Function Hapus Task Selesai
const clearCompletedTasks = () => {
  if (completedCount.value === 0) return alert("Belum ada task yang selesai.")
  if (confirm("Hapus semua task yang sudah selesai?")) {
    tasks.value = tasks.value.filter(task => !task.isDone)
  }
}

// --- COMPUTED PROPERTIES ---
const completedCount = computed(() => tasks.value.filter(t => t.isDone).length)
const totalCount = computed(() => tasks.value.length)
const pendingCount = computed(() => totalCount.value - completedCount.value)
const progressPercentage = computed(() => {
  return totalCount.value ? Math.round((completedCount.value / totalCount.value) * 100) : 0
})
</script>

<template>
  <!-- Main Spatial Viewport Wrapper -->
  <div 
    class="relative min-h-screen flex items-center justify-center p-4 sm:p-6 font-sans text-slate-900 select-none overflow-hidden transition-all duration-700"
    :class="`bg-gradient-to-br ${theme.bgGradient}`"
  >
    
    <!-- Spatial Dynamic Background Orbs -->
    <div :class="`absolute -top-10 -left-10 w-96 h-96 rounded-full blur-[100px] pointer-events-none animate-pulse ${theme.orbColors[0]}`"></div>
    <div :class="`absolute -bottom-10 -right-10 w-[30rem] h-[30rem] rounded-full blur-[120px] pointer-events-none ${theme.orbColors[1]}`"></div>
    <div :class="`absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-80 h-80 rounded-full blur-[90px] pointer-events-none ${theme.orbColors[2]}`"></div>

    <!-- Main Spatial Glass Container -->
    <main 
      class="relative w-full max-w-md backdrop-blur-2xl rounded-[2.5rem] p-7 shadow-[0_20px_50px_rgba(186,230,253,0.3)] border border-white/60 my-auto flex flex-col justify-between min-h-[580px] transition-all duration-500"
      :class="theme.glassOpacity"
    >
      
      <!-- Ambient Glow Overlay -->
      <div class="absolute inset-0 rounded-[2.5rem] bg-gradient-to-b from-white/40 via-transparent to-pink-100/20 pointer-events-none"></div>

      <!-- CONTENT CONTAINER -->
      <div class="relative z-10">
        <!-- Header Section -->
        <header class="mb-5 text-center">
          <span class="inline-block px-3.5 py-1 bg-white/50 backdrop-blur-md text-slate-800 font-bold rounded-full text-[11px] tracking-wider uppercase mb-2 border border-white/80 shadow-sm">
            Yorda aja
          </span>
          <h1 class="text-3xl font-extrabold tracking-tight text-slate-900 drop-shadow-sm">
            Taskify <span :class="theme.accentColor">Vue</span>
          </h1>
          <p class="text-xs font-bold text-slate-700 mt-1">I always loving u stranger.</p>
        </header>

        <!-- ==================== HALAMAN 1: TASKS VIEW ==================== -->
        <section v-if="activeTab === 'tasks'">
          <!-- Progress Metric Widget -->
          <div class="mb-4 p-4 rounded-2xl bg-white/40 backdrop-blur-md border border-white/70 flex items-center justify-between text-xs text-slate-900 shadow-inner">
            <div>
              <span class="font-extrabold text-slate-900">Progres Hari Ini</span>
              <p class="text-slate-800 font-bold mt-0.5">{{ completedCount }} dari {{ totalCount }} selesai</p>
            </div>
            <div class="text-xs font-black text-slate-900 bg-sky-200/80 backdrop-blur-sm px-3 py-1.5 rounded-xl border border-white/60 shadow-sm">
              {{ progressPercentage }}%
            </div>
          </div>

          <!-- Form Input Task -->
          <form @submit.prevent="addTask" class="flex items-center gap-2 mb-4">
            <input 
              v-model="taskInput"
              type="text" 
              placeholder="Tulis rencana atau task baru..." 
              class="flex-1 bg-white/50 backdrop-blur-md border border-white/80 focus:border-cyan-400 focus:bg-white/70 text-slate-900 font-bold text-sm rounded-2xl px-4 py-3 outline-none transition-all duration-300 placeholder:text-slate-500 shadow-inner"
            />
            <button 
              type="submit" 
              class="bg-gradient-to-r from-red-300 to-green-400 hover:from-blue-400 hover:to-yellow-500 active:scale-95 text-slate-900 font-black text-sm px-4 py-3 rounded-2xl shadow-[0_8px_20px_rgba(244,114,182,0.3)] border border-white/60 transition-all duration-200 shrink-0"
            >
              + Tambah
            </button>
          </form>

          <!-- Bulk Actions -->
          <div v-if="tasks.length > 0" class="flex items-center justify-end gap-2 mb-3 px-1">
            <button 
              v-if="completedCount > 0"
              @click="clearCompletedTasks"
              class="text-[11px] font-bold text-slate-700 hover:text-emerald-700 bg-white/30 hover:bg-white/50 border border-white/60 px-2.5 py-1 rounded-xl transition-all duration-200 backdrop-blur-sm"
            >
              ✓ Hapus Selesai
            </button>
            <button 
              @click="clearAllTasks"
              class="text-[11px] font-bold text-slate-700 hover:text-red-600 bg-white/30 hover:bg-white/50 border border-white/60 px-2.5 py-1 rounded-xl transition-all duration-200 backdrop-blur-sm"
            >
              ✕ Hapus Semua
            </button>
          </div>

          <!-- Task List Area -->
          <ul class="space-y-3 max-h-[28vh] overflow-y-auto pr-1">
            <li 
              v-for="task in tasks" 
              :key="task.id"
              class="flex items-center justify-between p-3.5 bg-white/45 backdrop-blur-md rounded-2xl border border-white/70 shadow-[0_4px_15px_rgba(0,0,0,0.03)] transition-all duration-200 hover:bg-white/60 hover:scale-[1.01]"
            >
              <div class="flex items-center gap-3 overflow-hidden cursor-pointer" @click="toggleTask(task)">
                <div 
                  class="w-5 h-5 rounded-lg border-2 transition-all duration-200 flex items-center justify-center shrink-0 shadow-sm"
                  :class="task.isDone ? 'bg-sky-300 border-sky-400 text-slate-900' : 'border-slate-800 bg-white/60'"
                >
                  <svg v-if="task.isDone" class="w-4 h-4 stroke-slate-900 stroke-[3]" viewBox="0 0 24 24" fill="none" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="20 6 9 17 4 12"></polyline>
                  </svg>
                </div>
                <span 
                  class="text-sm transition-all duration-200 font-bold truncate select-none"
                  :class="task.isDone ? 'line-through text-slate-500' : 'text-slate-900'"
                >
                  {{ task.title }}
                </span>
              </div>
              <button 
                @click="removeTask(task.id)"
                class="text-slate-700 hover:text-red-600 p-1.5 rounded-xl hover:bg-pink-200/50 transition-all duration-200 font-bold shrink-0 ml-2"
                title="Hapus Task"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                </svg>
              </button>
            </li>
            <li v-if="tasks.length === 0" class="text-center py-8 text-slate-800 font-bold text-sm bg-white/20 rounded-2xl border border-white/40">
              Belum ada kegiatan. Nikmati momen tenangmu! ☁️✨
            </li>
          </ul>
        </section>

        <!-- ==================== HALAMAN 2: ANALYTICS VIEW ==================== -->
        <section v-else-if="activeTab === 'analytics'" class="space-y-4">
          <div class="p-4 bg-white/40 backdrop-blur-md rounded-2xl border border-white/70 shadow-sm text-center">
            <h2 class="text-xs font-bold text-slate-700 uppercase tracking-wider mb-1">Total Progres</h2>
            <div class="text-4xl font-black text-slate-900 my-1">{{ progressPercentage }}%</div>
            <p class="text-xs font-semibold text-slate-700">Tingkat Penyelesaian Task</p>
          </div>
          <div class="grid grid-cols-2 gap-3">
            <div class="p-3.5 bg-sky-200/50 backdrop-blur-md rounded-2xl border border-white/70 text-center">
              <span class="text-xs font-extrabold text-slate-800">Selesai</span>
              <p class="text-2xl font-black text-slate-900 mt-1">{{ completedCount }}</p>
            </div>
            <div class="p-3.5 bg-green-200/50 backdrop-blur-md rounded-2xl border border-white/70 text-center">
              <span class="text-xs font-extrabold text-slate-800">Tertunda</span>
              <p class="text-2xl font-black text-slate-900 mt-1">{{ pendingCount }}</p>
            </div>
          </div>
          <div class="p-4 bg-white/40 backdrop-blur-md rounded-2xl border border-white/70">
            <span class="text-xs font-extrabold text-slate-900 mb-2 block">Statistik Visual</span>
            <div class="w-full h-3 bg-white/60 rounded-full overflow-hidden p-0.5 border border-white/80">
              <div 
                class="h-full bg-gradient-to-r from-sky-300 to-pink-400 rounded-full transition-all duration-500"
                :style="{ width: progressPercentage + '%' }"
              ></div>
            </div>
          </div>
        </section>

        <!-- ==================== HALAMAN 3: SETTINGS VIEW (BARU) ==================== -->
        <section v-else-if="activeTab === 'settings'" class="space-y-4">
          
          <!-- Background Gradient Picker -->
          <div class="p-4 bg-white/40 backdrop-blur-md rounded-2xl border border-white/70 shadow-sm">
            <label class="text-xs font-extrabold text-slate-800 uppercase tracking-wider mb-3 block">🎨 Background Tema</label>
            <div class="grid grid-cols-1 gap-2">
              <button 
                v-for="opt in bgOptions" 
                :key="opt.value"
                @click="theme.bgGradient = opt.value"
                class="flex items-center justify-between px-3 py-2.5 rounded-xl text-xs font-bold transition-all duration-200 border"
                :class="theme.bgGradient === opt.value 
                  ? 'bg-white/70 border-slate-400 text-slate-900 shadow-sm scale-[1.02]' 
                  : 'bg-white/20 border-white/40 text-slate-600 hover:bg-white/40'"
              >
                <span>{{ opt.label }}</span>
                <div v-if="theme.bgGradient === opt.value" class="w-2 h-2 rounded-full bg-slate-900"></div>
              </button>
            </div>
          </div>

          <!-- Glass Opacity Picker -->
          <div class="p-4 bg-white/40 backdrop-blur-md rounded-2xl border border-white/70 shadow-sm">
            <label class="text-xs font-extrabold text-slate-800 uppercase tracking-wider mb-3 block">💎 Glass Opacity</label>
            <div class="flex gap-2">
              <button 
                v-for="opt in glassOptions" 
                :key="opt.value"
                @click="theme.glassOpacity = opt.value"
                class="flex-1 py-2.5 rounded-xl text-[11px] font-bold transition-all duration-200 border"
                :class="theme.glassOpacity === opt.value 
                  ? 'bg-white/70 border-slate-400 text-slate-900 shadow-sm' 
                  : 'bg-white/20 border-white/40 text-slate-600 hover:bg-white/40'"
              >
                {{ opt.label }}
              </button>
            </div>
          </div>

          <!-- Accent Color Picker -->
          <div class="p-4 bg-white/40 backdrop-blur-md rounded-2xl border border-white/70 shadow-sm">
            <label class="text-xs font-extrabold text-slate-800 uppercase tracking-wider mb-3 block">✨ Accent Color</label>
            <div class="flex gap-2">
              <button 
                v-for="opt in accentOptions" 
                :key="opt.value"
                @click="theme.accentColor = opt.value"
                class="flex-1 py-2.5 rounded-xl text-[11px] font-bold transition-all duration-200 border"
                :class="[
                  opt.value,
                  theme.accentColor === opt.value 
                    ? 'bg-white/70 border-slate-400 shadow-sm ring-2 ring-offset-1 ring-offset-transparent ring-slate-300' 
                    : 'bg-white/20 border-white/40 text-slate-500 hover:bg-white/40'
                ]"
              >
                {{ opt.label }}
              </button>
            </div>
          </div>

          <!-- Reset Button -->
          <button 
            @click="resetTheme"
            class="w-full py-3 rounded-2xl text-xs font-bold text-slate-600 bg-white/20 border border-white/40 hover:bg-red-100/50 hover:text-red-600 hover:border-red-200 transition-all duration-200"
          >
            ↺ Kembalikan ke Default
          </button>
        </section>
      </div>

      <!-- INTERACTIVE NAVIGATION BAR (BOTTOM TABS) -->
      <nav class="relative z-10 mt-6 pt-3 border-t border-white/40 flex items-center justify-around">
        <!-- Tab: Tasks -->
        <button 
          @click="activeTab = 'tasks'"
          class="flex items-center gap-2 px-4 py-2 rounded-2xl font-extrabold text-xs transition-all duration-300"
          :class="activeTab === 'tasks' ? 'bg-white/70 shadow-md text-slate-900 border border-white/80 scale-105' : 'text-slate-700 hover:bg-white/30'"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
          </svg>
          Tasks
        </button>

        <!-- Tab: Analytics -->
        <button 
          @click="activeTab = 'analytics'"
          class="flex items-center gap-2 px-4 py-2 rounded-2xl font-extrabold text-xs transition-all duration-300"
          :class="activeTab === 'analytics' ? 'bg-white/70 shadow-md text-slate-900 border border-white/80 scale-105' : 'text-slate-700 hover:bg-white/30'"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 002 2h2a2 2 0 002-2z" />
          </svg>
          Statistik
        </button>

        <!-- Tab: Settings (BARU) -->
        <button 
          @click="activeTab = 'settings'"
          class="flex items-center gap-2 px-4 py-2 rounded-2xl font-extrabold text-xs transition-all duration-300"
          :class="activeTab === 'settings' ? 'bg-white/70 shadow-md text-slate-900 border border-white/80 scale-105' : 'text-slate-700 hover:bg-white/30'"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.066 2.573c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.573 1.066c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.066-2.573c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
          </svg>
          Setting
        </button>
      </nav>

    </main>
  </div>
</template>