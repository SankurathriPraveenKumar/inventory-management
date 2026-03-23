---
name: vue-saas-redesign
description: Redesigns a Vue 3 application's UI into a modern SaaS-style interface with a vertical navigation sidebar, consistent spacing, and a polished professional look. Use this skill when asked to redesign, modernize, or apply a SaaS layout to a Vue 3 app.
---

# Vue 3 SaaS UI Redesign

This skill transforms a Vue 3 application from a top-nav layout into a modern SaaS-style interface with a vertical sidebar, polished design system, and professional spacing. Follow every phase in order.

---

## When to Use This Skill

Invoke this skill when the user asks to:
- Redesign the UI into a sidebar layout
- Make the app look more like a modern SaaS product
- Replace a top navigation bar with a vertical nav
- Apply a consistent, professional design system

---

## Phase 1 — Audit the Existing App

Before writing a single line, read these files to understand the current structure:

1. **`src/App.vue`** — The root layout. Understand:
   - How the top nav is structured (router-links, logo, profile, etc.)
   - What global components are rendered (FilterBar, modals, etc.)
   - What global CSS is defined (the `<style>` block — note every class)
   - What composables are used (`useAuth`, `useI18n`, etc.)

2. **`src/main.js`** — All routes and their component names.

3. **Two or three view files** from `src/views/` — Note how they use `.page-header`, `.stats-grid`, `.card`, `.table-container`, etc. so you know which global classes to preserve.

4. **Any composables** used in App.vue (`useFilters`, `useAuth`, `useI18n`).

Document what you find:
- List of all nav routes (path + label)
- List of global CSS class names in App.vue that views depend on
- Any profile/user menu components
- Any filter bars or global UI elements
- Color variables in use

**Do not start implementing until the audit is complete.**

---

## Phase 2 — Choose a Sidebar Variant

Present the user with two options if they have not already specified one:

### Variant A — Dark Sidebar (Recommended for data-heavy apps)
```
┌─────────────────────────────────────────────────┐
│ ████████████ │                                   │
│ ████ Logo ██ │  Page Title                       │
│ ████████████ │  ─────────────────────────────    │
│              │                                   │
│  Overview    │  ┌──────┐ ┌──────┐ ┌──────┐      │
│  Inventory   │  │ KPI  │ │ KPI  │ │ KPI  │      │
│  Orders      │  └──────┘ └──────┘ └──────┘      │
│  Finance     │                                   │
│  Demand      │  ┌────────────────────────────┐   │
│  Reports     │  │         Table              │   │
│              │  └────────────────────────────┘   │
│ ─────────── │                                   │
│  User Name   │                                   │
└─────────────────────────────────────────────────┘
Sidebar: dark (#0f172a), 240px
Content: light (#f8fafc)
```

### Variant B — Light Sidebar
```
Sidebar: white (#ffffff) with right border (#e2e8f0), 240px
Content: light gray (#f8fafc)
Nav items: slate text with blue active state
```

---

## Phase 3 — Design System Tokens

Apply these exact values consistently throughout the redesign. Do not invent new values — only use tokens from this list.

### Colors

```css
/* Sidebar — Dark Variant */
--sidebar-bg:           #0f172a;
--sidebar-border:       #1e293b;
--sidebar-text:         #94a3b8;
--sidebar-text-hover:   #f1f5f9;
--sidebar-active-bg:    rgba(37, 99, 235, 0.15);
--sidebar-active-text:  #60a5fa;
--sidebar-active-bar:   #2563eb;
--sidebar-section-label:#475569;
--sidebar-icon-muted:   #64748b;

/* Sidebar — Light Variant */
--sidebar-bg:           #ffffff;
--sidebar-border:       #e2e8f0;
--sidebar-text:         #64748b;
--sidebar-text-hover:   #0f172a;
--sidebar-active-bg:    #eff6ff;
--sidebar-active-text:  #2563eb;
--sidebar-active-bar:   #2563eb;
--sidebar-section-label:#94a3b8;

/* Content Area */
--content-bg:           #f8fafc;
--content-border:       #e2e8f0;
--card-bg:              #ffffff;
--card-border:          #e2e8f0;
--card-shadow:          0 1px 3px rgba(0,0,0,0.04), 0 1px 2px rgba(0,0,0,0.03);
--card-shadow-hover:    0 4px 12px rgba(0,0,0,0.08);

/* Text */
--text-primary:         #0f172a;
--text-secondary:       #475569;
--text-muted:           #94a3b8;
--text-link:            #2563eb;

/* Accent */
--accent:               #2563eb;
--accent-hover:         #1d4ed8;
--accent-light:         #eff6ff;

/* Status */
--success:              #059669;
--success-bg:           #d1fae5;
--warning:              #d97706;
--warning-bg:           #fef3c7;
--danger:               #dc2626;
--danger-bg:            #fee2e2;
--info:                 #2563eb;
--info-bg:              #dbeafe;
```

### Spacing Scale

```css
--space-1:  4px;
--space-2:  8px;
--space-3:  12px;
--space-4:  16px;
--space-5:  20px;
--space-6:  24px;
--space-8:  32px;
--space-10: 40px;
--space-12: 48px;
```

### Typography

```css
/* Logo */
font-size: 15px; font-weight: 700; letter-spacing: -0.01em;

/* Nav items */
font-size: 13.5px; font-weight: 500;

/* Section labels */
font-size: 10.5px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.08em;

/* Page title */
font-size: 1.5rem (24px); font-weight: 700; letter-spacing: -0.025em;

/* Page subtitle */
font-size: 0.875rem (14px); color: var(--text-muted);

/* Card title */
font-size: 1rem (16px); font-weight: 600;

/* Table header */
font-size: 0.7rem (11.2px); font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em;

/* Table body */
font-size: 0.875rem (14px);

/* Stat value */
font-size: 2rem (32px); font-weight: 700; letter-spacing: -0.025em;

/* Stat label */
font-size: 0.75rem (12px); font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em;
```

### Sizing

```css
--sidebar-width:     240px;
--sidebar-collapsed: 64px;   /* If collapse is implemented */
--topbar-height:     0px;    /* No top bar in sidebar layout */
--nav-item-height:   36px;
--nav-item-radius:   6px;
--card-radius:       10px;
--badge-radius:      6px;
--button-radius:     6px;
--input-radius:      6px;
--content-max-width: 1400px;
--content-padding:   24px 28px;
```

---

## Phase 4 — Implement the New Layout in `App.vue`

### 4a. New Template Structure

Replace the entire template with this structure (adapt labels and components from your audit):

```vue
<template>
  <div class="app-shell">

    <!-- ── Sidebar ── -->
    <aside class="sidebar">
      <div class="sidebar-inner">

        <!-- Logo / Brand -->
        <div class="sidebar-brand">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-sub">{{ t('nav.subtitle') }}</span>
        </div>

        <!-- Primary Navigation -->
        <nav class="sidebar-nav">
          <!-- Optional: section label -->
          <!-- <div class="nav-section-label">Main</div> -->

          <router-link
            v-for="link in navLinks"
            :key="link.path"
            :to="link.path"
            :class="['nav-item', { active: isActive(link) }]"
          >
            <span class="nav-icon">{{ link.icon }}</span>
            <span class="nav-label">{{ link.label }}</span>
          </router-link>
        </nav>

        <!-- Sidebar Footer: user + language -->
        <div class="sidebar-footer">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>

      </div>
    </aside>

    <!-- ── Main Area ── -->
    <div class="main-area">
      <!-- Optional top strip for filters — only show on relevant pages -->
      <div class="topbar" v-if="showFilterBar">
        <FilterBar />
      </div>

      <main class="content">
        <router-view />
      </main>
    </div>

    <!-- Modals — unchanged from original -->
    <ProfileDetailsModal :is-open="showProfileDetails" @close="showProfileDetails = false" />
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
```

### 4b. Script Additions

Add `navLinks` data and `isActive` helper to `setup()`. The `navLinks` array is built from the routes you found in Phase 1. Use single characters or short Unicode symbols as icons (no emoji, per project style):

```javascript
import { useRoute } from 'vue-router'

// Inside setup():
const route = useRoute()

// Build from your audited route list. Icons: use simple text symbols or SVG paths.
const navLinks = [
  { path: '/',           label: t('nav.overview'),       icon: '◈' },
  { path: '/inventory',  label: t('nav.inventory'),      icon: '▦' },
  { path: '/orders',     label: t('nav.orders'),         icon: '≡' },
  { path: '/spending',   label: t('nav.finance'),        icon: '◎' },
  { path: '/demand',     label: t('nav.demandForecast'), icon: '↗' },
  { path: '/reports',    label: 'Reports',               icon: '▤' },
  { path: '/restocking', label: t('nav.restocking'),     icon: '⟳' },
  // Add any new routes here
]

// Active state: exact match for '/', prefix match for sub-routes
const isActive = (link) => {
  if (link.path === '/') return route.path === '/'
  return route.path.startsWith(link.path)
}

// Control whether FilterBar shows (hide it on pages that don't use filters)
const noFilterRoutes = ['/reports', '/restocking']
const showFilterBar = computed(() => !noFilterRoutes.includes(route.path))

// Return these alongside existing returns:
// navLinks, isActive, showFilterBar
```

### 4c. Global CSS — Full Replacement

Replace the entire `<style>` block with the following. **Preserve all badge, table, stat-card, card, loading, and error classes verbatim** — views depend on them. Only replace the layout wrapper classes.

```css
/* ── Reset ── */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f8fafc;
  color: #0f172a;
  -webkit-font-smoothing: antialiased;
}

/* ── App shell: sidebar + content side-by-side ── */
.app-shell {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 240px;
  flex-shrink: 0;
  background: #0f172a;           /* dark variant */
  border-right: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 50;
  overflow-y: auto;
}

.sidebar-inner {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 0;
}

/* Brand */
.sidebar-brand {
  padding: 20px 16px 16px;
  border-bottom: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.brand-name {
  font-size: 14px;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-sub {
  font-size: 11px;
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Navigation */
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
  padding: 12px 8px 6px;
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
  transition: background 0.15s ease, color 0.15s ease;
  cursor: pointer;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.06);
  color: #f1f5f9;
}

.nav-item.active {
  background: rgba(37, 99, 235, 0.18);
  color: #60a5fa;
  position: relative;
}

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

.nav-icon {
  font-size: 14px;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
  opacity: 0.8;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Sidebar footer */
.sidebar-footer {
  padding: 12px 8px;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

/* ── Main area ── */
.main-area {
  flex: 1;
  margin-left: 240px;          /* offset for fixed sidebar */
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

/* Topbar (filter bar strip) */
.topbar {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 0 28px;
  position: sticky;
  top: 0;
  z-index: 40;
}

/* Content */
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

/* ── Loading / Error ── */
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
```

---

## Phase 5 — Update the FilterBar Component

The FilterBar no longer lives in the top nav. Verify `FilterBar.vue` does not have hardcoded height or margin assumptions for a top nav context. If it does:

- Remove any `position: sticky; top: 70px` or fixed-height assumptions
- Set its internal padding to `12px 0` so it sits flush inside `.topbar`
- Ensure it still uses the `useFilters` composable unchanged

---

## Phase 6 — Per-View Adjustments

After updating `App.vue`, check each view file for these issues:

### 6a. Remove hardcoded top offsets
If any view has `margin-top` compensating for a top nav, remove it.

### 6b. Verify `.page-header` usage
Every view should start with:
```vue
<div class="page-header">
  <h2>{{ t('section.title') }}</h2>
  <p>{{ t('section.description') }}</p>
</div>
```

### 6c. Verify `.stats-grid` and `.card` are used correctly
These classes now use the updated global styles. No view-level overrides should be needed.

---

## Phase 7 — Optional Polish

Apply these upgrades if the user wants a more finished look:

### Sidebar section groupings
If there are 6+ nav items, group them with section labels:
```vue
<div class="nav-section-label">Analytics</div>
<router-link ...>Demand Forecast</router-link>
<router-link ...>Reports</router-link>
```

### Collapsed sidebar (mobile/narrow)
Add a toggle button and CSS media query:
```css
@media (max-width: 768px) {
  .sidebar { width: 64px; }
  .brand-sub, .nav-label, .brand-name { display: none; }
  .main-area { margin-left: 64px; }
}
```

### Subtle card hover shadow
Already included in the CSS above via `.card:hover`. No extra work needed.

### Active nav indicator
The left-border `::before` pseudo-element is already included. If the user prefers a filled pill instead, change `.nav-item.active` to:
```css
.nav-item.active {
  background: #2563eb;
  color: #ffffff;
}
.nav-item.active::before { display: none; }
```

---

## Phase 8 — Verification Checklist

Run through every item before declaring the redesign done:

**Layout:**
- [ ] Sidebar is 240px wide and full-height
- [ ] Sidebar does not scroll with the page (fixed/sticky)
- [ ] Main content area fills remaining width correctly
- [ ] No horizontal overflow on any page
- [ ] Page content has consistent 28px padding on all sides

**Navigation:**
- [ ] Every route from `main.js` has a nav link in the sidebar
- [ ] Active link is highlighted correctly on each page
- [ ] Exact-match active state only highlights `/` when on the home page
- [ ] Logo / brand text visible in sidebar header

**Design consistency:**
- [ ] All stat cards use `.stat-card` with proper color modifiers
- [ ] All section headings use `.card-title`
- [ ] All status indicators use `.badge` with correct class (`success`, `danger`, etc.)
- [ ] Tables use `thead`/`tbody`/`th`/`td` global styles (no inline overrides)
- [ ] Loading states use `.loading` class
- [ ] Error states use `.error` class

**Functional:**
- [ ] FilterBar still appears and works on filterable pages
- [ ] FilterBar hidden on non-filterable pages (reports, restocking, etc.)
- [ ] Profile menu still opens from sidebar footer
- [ ] Language switcher still works from sidebar footer
- [ ] All modals still open and close correctly
- [ ] Router navigation works — clicking each nav item loads the correct view
- [ ] Browser back/forward still works

**Visual:**
- [ ] No leftover top-nav CSS (`.top-nav`, `.nav-container` classes gone or unused)
- [ ] Sidebar background color is consistent end-to-end (no white gaps at bottom)
- [ ] Card borders and shadows are consistent across pages
- [ ] Typography scale is consistent (titles 24px, labels 12px, body 14px)

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `margin-left` on `.main-area` doesn't match sidebar width | Keep `margin-left: 240px` in sync with `.sidebar { width: 240px }` |
| FilterBar renders with wrong height inside `.topbar` | Set FilterBar's outer padding to `0`, let `.topbar` control vertical space |
| Router `active` class conflicts with Vue Router's automatic `.router-link-active` | Use `:class="['nav-item', { active: isActive(link) }]"` instead of relying on the automatic class |
| Sidebar overflows hidden content when height < viewport | Ensure `.sidebar { overflow-y: auto }` is set |
| Profile/language components look wrong in footer | Give `.sidebar-footer` `flex-wrap: wrap` if items are cramped |
| `position: fixed` sidebar causes content overlap in Firefox | Also set `position: sticky; height: 100vh` as a fallback |
| White flash between pages on dark sidebar | Set `body { background: #f8fafc }` on the content side, not globally |

---

## Key Reminders

- **Preserve all existing badge, card, table, loading, and error classes** — view files depend on them. Only replace layout wrapper classes.
- **Do not change route paths or component names** — only the nav UI changes.
- **Do not touch composables** (`useFilters`, `useAuth`, `useI18n`) — they are layout-agnostic.
- **Sidebar width (240px) must match margin-left on `.main-area`** — keep them in sync.
- **Test on the actual running app** after changes — use the browser at `http://localhost:3000`.
