<template>
  <div class="app-shell">

    <!-- ── Sidebar ── -->
    <aside :class="['sidebar', { collapsed: isCollapsed }]">
      <div class="sidebar-inner">

        <!-- Brand + collapse toggle -->
        <div class="sidebar-brand">
          <div class="brand-text">
            <span class="brand-name">{{ t('nav.companyName') }}</span>
            <span class="brand-sub">{{ t('nav.subtitle') }}</span>
          </div>
          <button
            class="sidebar-toggle"
            @click="toggleSidebar"
            :title="isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
          >
            <!-- Chevron points left when expanded, right when collapsed -->
            <svg viewBox="0 0 20 20" fill="currentColor" width="16" height="16">
              <path
                fill-rule="evenodd"
                :d="isCollapsed
                  ? 'M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z'
                  : 'M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z'"
                clip-rule="evenodd"
              />
            </svg>
          </button>
        </div>

        <!-- Navigation -->
        <nav class="sidebar-nav">
          <div class="nav-section-label">Main</div>
          <router-link
            v-for="link in mainLinks"
            :key="link.path"
            :to="link.path"
            :class="['nav-item', { active: isActive(link) }]"
            :title="isCollapsed ? link.label : ''"
          >
            <span class="nav-icon">{{ link.icon }}</span>
            <span class="nav-label">{{ link.label }}</span>
          </router-link>

          <div class="nav-section-label" style="margin-top: 8px;">Analytics</div>
          <router-link
            v-for="link in analyticsLinks"
            :key="link.path"
            :to="link.path"
            :class="['nav-item', { active: isActive(link) }]"
            :title="isCollapsed ? link.label : ''"
          >
            <span class="nav-icon">{{ link.icon }}</span>
            <span class="nav-label">{{ link.label }}</span>
          </router-link>
        </nav>

        <!-- Footer: language + profile -->
        <div class="sidebar-footer">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>

      </div>
    </aside>

    <!-- ── Main area — shifts left when sidebar collapses ── -->
    <div :class="['main-area', { collapsed: isCollapsed }]">
      <!-- Filter bar strip — hidden on pages that don't use filters -->
      <div class="topbar" v-if="showFilterBar">
        <FilterBar />
      </div>

      <main class="content">
        <router-view />
      </main>
    </div>

    <!-- Modals — unchanged -->
    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />
    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const route = useRoute()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Sidebar collapse state — auto-collapse on small screens
    const isCollapsed = ref(false)
    const toggleSidebar = () => { isCollapsed.value = !isCollapsed.value }

    const handleResize = () => {
      // Auto-collapse below 768px; auto-expand at or above it
      isCollapsed.value = window.innerWidth < 768
    }

    // Nav link groups
    const mainLinks = [
      { path: '/',          label: t('nav.overview'),       icon: '◈' },
      { path: '/inventory', label: t('nav.inventory'),      icon: '▦' },
      { path: '/orders',    label: t('nav.orders'),         icon: '≡' },
      { path: '/spending',  label: t('nav.finance'),        icon: '◎' },
    ]
    const analyticsLinks = [
      { path: '/demand',      label: t('nav.demandForecast'), icon: '↗' },
      { path: '/reports',     label: 'Reports',               icon: '▤' },
      { path: '/restocking',  label: t('nav.restocking'),     icon: '⟳' },
    ]

    // Exact match for root, prefix match for all other routes
    const isActive = (link) => {
      if (link.path === '/') return route.path === '/'
      return route.path.startsWith(link.path)
    }

    // Hide the filter bar on pages that don't use global filters
    const noFilterRoutes = ['/reports', '/restocking']
    const showFilterBar = computed(() => !noFilterRoutes.includes(route.path))

    // Tasks — merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) currentUser.value.tasks.splice(index, 1)
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) apiTasks.value[index] = updatedTask
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(() => {
      loadTasks()
      // Set initial collapse state based on viewport width
      handleResize()
      window.addEventListener('resize', handleResize)
    })

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    return {
      t,
      mainLinks,
      analyticsLinks,
      isActive,
      showFilterBar,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isCollapsed,
      toggleSidebar
    }
  }
}
</script>

<style>
/* ── Reset ── */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f8fafc;
  color: #0f172a;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── App shell ── */
.app-shell {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 240px;
  flex-shrink: 0;
  background: #0f172a;
  border-right: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 50;
  overflow-y: auto;
  overflow-x: hidden;
  /* Smooth width transition for collapse/expand */
  transition: width 0.25s ease;
}

/* ── Collapsed sidebar: icons only ── */
.sidebar.collapsed {
  width: 56px;
}

.sidebar-inner {
  display: flex;
  flex-direction: column;
  min-height: 100%;
  /* Prevent inner content from wrapping during transition */
  width: 240px;
}

.sidebar-brand {
  padding: 14px 10px 14px 16px;
  border-bottom: 1px solid #1e293b;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-shrink: 0;
  min-height: 64px;
}

.brand-text {
  display: flex;
  flex-direction: column;
  gap: 3px;
  overflow: hidden;
  transition: opacity 0.15s ease, width 0.25s ease;
}

/* Hide brand text when collapsed */
.sidebar.collapsed .brand-text {
  opacity: 0;
  width: 0;
  pointer-events: none;
}

.brand-name {
  font-size: 14px;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: -0.01em;
  white-space: nowrap;
}

.brand-sub {
  font-size: 11px;
  color: #475569;
  white-space: nowrap;
}

/* Collapse/expand toggle button */
.sidebar-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  border: 1px solid #1e293b;
  border-radius: 6px;
  background: transparent;
  color: #475569;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease, border-color 0.15s ease;
}

.sidebar-toggle:hover {
  background: rgba(255, 255, 255, 0.08);
  color: #94a3b8;
  border-color: #334155;
}

/* Center the toggle when collapsed */
.sidebar.collapsed .sidebar-brand {
  justify-content: center;
  padding: 14px 0;
}

.sidebar-nav {
  flex: 1;
  padding: 12px 8px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-section-label {
  font-size: 10.5px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #475569;
  padding: 10px 8px 5px;
  white-space: nowrap;
  overflow: hidden;
  transition: opacity 0.15s ease;
}

/* Hide section labels when collapsed */
.sidebar.collapsed .nav-section-label {
  opacity: 0;
  height: 0;
  padding: 0;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 10px;
  height: 36px;
  border-radius: 6px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 13.5px;
  font-weight: 500;
  transition: background 0.15s ease, color 0.15s ease, padding 0.25s ease;
  position: relative;
  white-space: nowrap;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.06);
  color: #f1f5f9;
}

.nav-item.active {
  background: rgba(37, 99, 235, 0.18);
  color: #60a5fa;
}

/* Left accent bar on active item */
.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: #2563eb;
}

/* Center icons when collapsed */
.sidebar.collapsed .nav-item {
  padding: 0;
  justify-content: center;
}

.nav-icon {
  font-size: 16px;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
  opacity: 0.85;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: opacity 0.15s ease;
}

/* Hide nav labels when collapsed */
.sidebar.collapsed .nav-label {
  opacity: 0;
  width: 0;
  overflow: hidden;
}

.sidebar-footer {
  padding: 12px 8px;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-shrink: 0;
  transition: padding 0.25s ease;
}

/* Stack footer items vertically when collapsed */
.sidebar.collapsed .sidebar-footer {
  flex-direction: column;
  justify-content: center;
  padding: 12px 0;
  gap: 4px;
}

/* ── Main area ── */
.main-area {
  flex: 1;
  margin-left: 240px;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  /* Smooth shift when sidebar collapses */
  transition: margin-left 0.25s ease;
}

.main-area.collapsed {
  margin-left: 56px;
}

/* Filter bar strip */
.topbar {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  position: sticky;
  top: 0;
  z-index: 40;
}

/* Page content */
.content {
  flex: 1;
  padding: 28px;
  max-width: 1400px;
  width: 100%;
}

/* ── Page header ── */
.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  margin-bottom: 4px;
}

.page-header p {
  color: #64748b;
  font-size: 0.875rem;
}

/* ── Stat cards ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
  margin-bottom: 20px;
}

.stat-card {
  background: #ffffff;
  padding: 20px;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0,0,0,0.04);
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  line-height: 1;
}

.stat-card.success .stat-value { color: #059669; }
.stat-card.warning .stat-value { color: #d97706; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: #2563eb; }

/* ── Cards ── */
.card {
  background: #ffffff;
  border-radius: 10px;
  padding: 20px;
  border: 1px solid #e2e8f0;
  margin-bottom: 20px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.04);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 14px;
  border-bottom: 1px solid #f1f5f9;
}

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: #0f172a;
  letter-spacing: -0.01em;
}

/* ── Tables ── */
.table-container { overflow-x: auto; }

table { width: 100%; border-collapse: collapse; }

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 8px 12px;
  font-weight: 700;
  color: #64748b;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  white-space: nowrap;
}

td {
  padding: 10px 12px;
  border-bottom: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr:last-child td { border-bottom: none; }
tbody tr { transition: background-color 0.1s ease; }
tbody tr:hover { background: #f8fafc; }

/* ── Badges ── */
.badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  border-radius: 6px;
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  white-space: nowrap;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #fee2e2; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fee2e2; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fee2e2; color: #991b1b; }
.badge.medium     { background: #fef3c7; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── States ── */
.loading {
  text-align: center;
  padding: 48px;
  color: #94a3b8;
  font-size: 0.875rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 14px 16px;
  border-radius: 8px;
  margin: 12px 0;
  font-size: 0.875rem;
}
</style>
