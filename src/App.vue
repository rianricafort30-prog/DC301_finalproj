<script setup>
import { computed, onMounted, ref, watch } from 'vue'

import {
  Activity,
  CalendarDays,
  CheckCircle2,
  ChevronRight,
  CirclePlus,
  ClipboardList,
  Clock3,
  Dumbbell,
  Home,
  MapPin,
  Pencil,
  Search,
  Trash2,
  Users,
  Volleyball,
  X
} from 'lucide-vue-next'

const STORAGE_KEY = 'volleytrain-sessions-v1'

const activeTab = ref('dashboard')
const searchText = ref('')
const showForm = ref(false)
const editingId = ref(null)
const message = ref('')
const sessions = ref([])

const emptyForm = () => ({
  title: '',
  date: '',
  time: '',
  coach: '',
  location: '',
  notes: '',
  status: 'Upcoming'
})

const form = ref(emptyForm())

// LOAD SAVED DATA
onMounted(() => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY)

    if (saved) {
      const parsed = JSON.parse(saved)

      if (Array.isArray(parsed)) {
        sessions.value = parsed
      }
    } else {
      sessions.value = [
        {
          id: Date.now(),
          title: 'Basic Spiking Training',
          date: new Date().toISOString().slice(0, 10),
          time: '16:00',
          coach: 'Coach Mark',
          location: 'Main Court',
          notes: 'Warm-up, approach, and spiking drills.',
          status: 'Upcoming'
        }
      ]
    }
  } catch (error) {
    sessions.value = []
  }
})

// SAVE DATA AUTOMATICALLY
watch(
  sessions,
  value => {
    localStorage.setItem(
      STORAGE_KEY,
      JSON.stringify(value)
    )
  },
  { deep: true }
)

// DASHBOARD STATISTICS
const upcomingCount = computed(() =>
  sessions.value.filter(
    session => session.status === 'Upcoming'
  ).length
)

const completedCount = computed(() =>
  sessions.value.filter(
    session => session.status === 'Completed'
  ).length
)

// SEARCH AND SORT SESSIONS
const filteredSessions = computed(() => {
  const query = searchText.value.trim().toLowerCase()

  return [...sessions.value]
    .filter(session => {
      return (
        !query ||
        [
          session.title,
          session.coach,
          session.location,
          session.status
        ].some(value =>
          (value || '').toLowerCase().includes(query)
        )
      )
    })
    .sort((a, b) =>
      `${a.date} ${a.time}`.localeCompare(
        `${b.date} ${b.time}`
      )
    )
})

// UPCOMING SESSIONS FOR DASHBOARD
const nextSessions = computed(() =>
  [...sessions.value]
    .filter(session => session.status === 'Upcoming')
    .sort((a, b) =>
      `${a.date} ${a.time}`.localeCompare(
        `${b.date} ${b.time}`
      )
    )
    .slice(0, 3)
)

// FORMAT DATE
function formatDate(value) {
  if (!value) return 'No date'

  const date = new Date(`${value}T00:00:00`)

  return date.toLocaleDateString(undefined, {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  })
}

// FORMAT TIME
function formatTime(value) {
  if (!value) return 'No time'

  const [hour, minute] = value.split(':').map(Number)
  const date = new Date()

  date.setHours(hour, minute, 0, 0)

  return date.toLocaleTimeString(undefined, {
    hour: 'numeric',
    minute: '2-digit'
  })
}

// OPEN ADD FORM
function openAdd() {
  editingId.value = null
  form.value = emptyForm()
  showForm.value = true
}

// OPEN EDIT FORM
function openEdit(session) {
  editingId.value = session.id
  form.value = { ...session }
  showForm.value = true
}

// CREATE OR UPDATE SESSION
function saveSession() {
  if (
    !form.value.title.trim() ||
    !form.value.date ||
    !form.value.time ||
    !form.value.coach.trim() ||
    !form.value.location.trim()
  ) {
    message.value =
      'Please complete the title, date, time, coach, and location.'

    setTimeout(() => {
      message.value = ''
    }, 3000)

    return
  }

  if (editingId.value !== null) {
    const index = sessions.value.findIndex(
      session => session.id === editingId.value
    )

    if (index !== -1) {
      sessions.value[index] = {
        ...form.value,
        id: editingId.value
      }
    }

    message.value = 'Training session updated.'
  } else {
    sessions.value.push({
      ...form.value,
      id: Date.now()
    })

    message.value = 'Training session added.'
  }

  showForm.value = false
  activeTab.value = 'sessions'

  setTimeout(() => {
    message.value = ''
  }, 2500)
}

// DELETE SESSION
function deleteSession(id) {
  if (!window.confirm('Delete this training session?')) {
    return
  }

  sessions.value = sessions.value.filter(
    session => session.id !== id
  )

  message.value = 'Training session deleted.'

  setTimeout(() => {
    message.value = ''
  }, 2500)
}

// UPDATE SESSION STATUS
function setStatus(session, status) {
  const index = sessions.value.findIndex(
    item => item.id === session.id
  )

  if (index !== -1) {
    sessions.value[index].status = status
  }
}
</script>

<template>
  <div class="app-shell">

    <!-- HEADER -->
    <header class="topbar">
      <div class="brand-mark">
        <Volleyball :size="25" />
      </div>

      <div class="brand-copy">
        <strong>VolleyTrain</strong>
        <span>TRAIN SMART. PLAY STRONG.</span>
      </div>

      <div class="topbar-pill">
        <span></span>
        Offline ready
      </div>
    </header>

    <main class="main-content">

      <!-- DASHBOARD -->
      <section v-if="activeTab === 'dashboard'" class="page">

        <div class="greeting">
          <div>
            <p class="eyebrow">YOUR TEAM WORKSPACE</p>

            <h1>
              Ready to <span>train?</span>
            </h1>

            <p class="muted">
              Plan your next session and keep improving.
            </p>
          </div>

          <div class="volleyball-art">
            <Volleyball :size="42" />
          </div>
        </div>

        <button
          class="primary-button wide"
          @click="openAdd"
        >
          <CirclePlus :size="19" />
          Add training session
        </button>

        <!-- STATISTICS -->
        <div class="stats-grid">

          <article class="stat-card">
            <div class="stat-icon purple">
              <ClipboardList :size="20" />
            </div>

            <p>Total sessions</p>
            <strong>{{ sessions.length }}</strong>
            <span>All training records</span>
          </article>

          <article class="stat-card">
            <div class="stat-icon orange">
              <Clock3 :size="20" />
            </div>

            <p>Upcoming</p>
            <strong>{{ upcomingCount }}</strong>
            <span>Still to be done</span>
          </article>

          <article class="stat-card">
            <div class="stat-icon green">
              <CheckCircle2 :size="20" />
            </div>

            <p>Completed</p>
            <strong>{{ completedCount }}</strong>
            <span>Sessions finished</span>
          </article>

        </div>

        <!-- UPCOMING SESSIONS -->
        <div class="section-heading">
          <div>
            <p class="eyebrow">KEEP THE MOMENTUM</p>
            <h2>Upcoming sessions</h2>
          </div>

          <button
            class="text-button"
            @click="activeTab = 'sessions'"
          >
            See all
            <ChevronRight :size="16" />
          </button>
        </div>

        <div
          v-if="nextSessions.length"
          class="session-list"
        >
          <article
            v-for="session in nextSessions"
            :key="session.id"
            class="session-card"
          >
            <div class="date-tile">
              <span>
                {{
                  new Date(`${session.date}T00:00:00`)
                    .toLocaleDateString(undefined, {
                      month: 'short'
                    })
                    .toUpperCase()
                }}
              </span>

              <strong>
                {{
                  new Date(`${session.date}T00:00:00`)
                    .getDate()
                }}
              </strong>
            </div>

            <div class="session-info">
              <strong>{{ session.title }}</strong>

              <span>
                <Clock3 :size="14" />
                {{ formatTime(session.time) }}
              </span>

              <span>
                <MapPin :size="14" />
                {{ session.location }}
              </span>
            </div>

            <button
              class="icon-button"
              aria-label="Edit session"
              @click="openEdit(session)"
            >
              <Pencil :size="17" />
            </button>
          </article>
        </div>

        <div v-else class="empty-state">
          <CalendarDays :size="30" />
          <strong>Nothing scheduled yet</strong>
          <span>
            Add a session to start planning your training.
          </span>
        </div>

        <div class="tip-card">
          <div class="tip-icon">
            <Dumbbell :size="20" />
          </div>

          <div>
            <strong>Consistency builds champions.</strong>

            <p>
              Schedule your sessions, track your progress,
              and show up ready.
            </p>
          </div>
        </div>

      </section>

      <!-- SESSION LIST -->
      <section v-else class="page">

        <div class="list-page-heading">
          <p class="eyebrow">YOUR TRAINING PLAN</p>
          <h1>Sessions</h1>
          <p class="muted">
            Manage every practice in one place.
          </p>
        </div>

        <button
          class="primary-button wide"
          @click="openAdd"
        >
          <CirclePlus :size="19" />
          Add training session
        </button>

        <!-- SEARCH -->
        <label class="search-box">
          <Search :size="18" />

          <input
            v-model="searchText"
            type="search"
            placeholder="Search sessions, coach, location..."
          />
        </label>

        <div class="list-summary">
          <strong>
            {{ filteredSessions.length }}
            session{{ filteredSessions.length === 1 ? '' : 's' }}
          </strong>

          <span>Saved on this device</span>
        </div>

        <div
          v-if="filteredSessions.length"
          class="session-list full-list"
        >
          <article
            v-for="session in filteredSessions"
            :key="session.id"
            class="session-card detailed"
          >
            <div class="session-card-top">
              <span
                class="status-chip"
                :class="session.status.toLowerCase()"
              >
                {{ session.status }}
              </span>

              <div class="card-actions">
                <button
                  class="icon-button"
                  aria-label="Edit session"
                  @click="openEdit(session)"
                >
                  <Pencil :size="17" />
                </button>

                <button
                  class="icon-button danger"
                  aria-label="Delete session"
                  @click="deleteSession(session.id)"
                >
                  <Trash2 :size="17" />
                </button>
              </div>
            </div>

            <h3>{{ session.title }}</h3>

            <div class="detail-line">
              <CalendarDays :size="16" />
              {{ formatDate(session.date) }} ·
              {{ formatTime(session.time) }}
            </div>

            <div class="detail-line">
              <Users :size="16" />
              {{ session.coach }}
            </div>

            <div class="detail-line">
              <MapPin :size="16" />
              {{ session.location }}
            </div>

            <p v-if="session.notes" class="notes">
              {{ session.notes }}
            </p>

            <div class="status-actions">
              <button
                v-if="session.status !== 'Completed'"
                class="complete-button"
                @click="setStatus(session, 'Completed')"
              >
                <CheckCircle2 :size="15" />
                Mark completed
              </button>

              <button
                v-else
                class="undo-button"
                @click="setStatus(session, 'Upcoming')"
              >
                Move to upcoming
              </button>
            </div>
          </article>
        </div>

        <div v-else class="empty-state">
          <ClipboardList :size="30" />
          <strong>No sessions found</strong>
          <span>
            Try another search or add a new training session.
          </span>
        </div>

      </section>
    </main>

    <!-- BOTTOM NAVIGATION -->
    <nav class="bottom-nav">
      <button
        :class="{ active: activeTab === 'dashboard' }"
        @click="activeTab = 'dashboard'"
      >
        <Home :size="21" />
        <span>Home</span>
      </button>

      <button
        :class="{ active: activeTab === 'sessions' }"
        @click="activeTab = 'sessions'"
      >
        <CalendarDays :size="21" />
        <span>Sessions</span>
      </button>
    </nav>

    <!-- SUCCESS MESSAGE -->
    <Transition name="toast">
      <div v-if="message" class="toast-message">
        {{ message }}
      </div>
    </Transition>

    <!-- ADD / EDIT FORM -->
    <div
      v-if="showForm"
      class="modal-backdrop"
      @click.self="showForm = false"
    >
      <section class="form-sheet">

        <div class="sheet-heading">
          <div>
            <p class="eyebrow">
              {{
                editingId === null
                  ? 'ADD TO YOUR PLAN'
                  : 'UPDATE YOUR PLAN'
              }}
            </p>

            <h2>
              {{
                editingId === null
                  ? 'New session'
                  : 'Edit session'
              }}
            </h2>
          </div>

          <button
            class="icon-button"
            aria-label="Close form"
            @click="showForm = false"
          >
            <X :size="21" />
          </button>
        </div>

        <form @submit.prevent="saveSession">

          <label>
            Training title *
            <input
              v-model="form.title"
              maxlength="80"
              required
              placeholder="e.g. Spiking drills"
            />
          </label>

          <div class="form-row">
            <label>
              Date *
              <input
                v-model="form.date"
                type="date"
                required
              />
            </label>

            <label>
              Time *
              <input
                v-model="form.time"
                type="time"
                required
              />
            </label>
          </div>

          <label>
            Coach name *
            <input
              v-model="form.coach"
              required
              placeholder="e.g. Coach Rian"
            />
          </label>

          <label>
            Training location *
            <input
              v-model="form.location"
              required
              placeholder="e.g. School gym"
            />
          </label>

          <label>
            Status
            <select v-model="form.status">
              <option>Upcoming</option>
              <option>Completed</option>
            </select>
          </label>

          <label>
            Notes (optional)
            <textarea
              v-model="form.notes"
              rows="3"
              placeholder="Drills, goals, or reminders..."
            ></textarea>
          </label>

          <div class="form-buttons">
            <button
              type="button"
              class="secondary-button"
              @click="showForm = false"
            >
              Cancel
            </button>

            <button type="submit" class="primary-button">
              <CheckCircle2 :size="18" />

              {{
                editingId === null
                  ? 'Save session'
                  : 'Update session'
              }}
            </button>
          </div>

        </form>
      </section>
    </div>

  </div>
</template>