# Meridian Analytics — Client Dashboard

A real-time SaaS analytics dashboard: revenue, user growth, sales, traffic,
devices, countries, activity feed, notifications, profile, and settings —
all backed by a mock REST layer with realistic loading and error states.

## Run it

```bash
npm install
npm run dev
```

Then open the printed local URL. `npm run build` produces a static
production build in `dist/`.

## Project structure

```
src/
  theme.js                    Design tokens (colors)
  Dashboard.jsx                Top-level page — wires sections together
  main.jsx                     React entry point
  api/
    mockApi.js                 Simulated REST endpoints (latency + failures)
  hooks/
    useSection.js              Fetch/loading/error/reload hook per section
  utils/
    format.js                  Number/date/currency formatting helpers
  components/
    Sidebar.jsx                Left nav, collapses to off-canvas on mobile
    Topbar.jsx                 Search, date range, refresh, notifications, profile
    KpiCard.jsx                Revenue / Users / Sales / Conversion stat cards
    Card.jsx                   Generic card shell + legend + chart tooltip
    NotificationsPanel.jsx     Bell dropdown
    ProfilePanel.jsx           Avatar dropdown
    SettingsPanel.jsx          Slide-over settings panel
    Skeletons.jsx              Loading skeleton placeholders
    ErrorState.jsx             Per-widget error banner with retry
  styles/
    dashboard.css              All styling (the "Instrument Panel" theme)
index.html
vite.config.js
package.json
```

## Wiring up a real backend

Every section fetches through `src/api/mockApi.js`. Each function
(`Api.kpis`, `Api.revenue`, `Api.sales`, etc.) currently returns a Promise
that resolves with generated data after a simulated delay, and occasionally
rejects to exercise the error states. Replace the body of each function with
a real `fetch("/api/...")` call — the rest of the app (skeletons, retry
buttons, auto-refresh) needs no changes since it only depends on the
Promise contract.

```js
// before
kpis: (range) => apiCall(() => ({ ...generated... }))

// after
kpis: (range) => fetch(`/api/kpis?range=${range}`).then(r => {
  if (!r.ok) throw new Error(`Request failed: ${r.status}`);
  return r.json();
})
```

## Notes

- **Auto-refresh**: toggle in the top bar; when on, every section silently
  re-fetches every 20 seconds without showing the loading skeleton.
- **Error handling**: each of the 8 data sections (KPIs, revenue, growth,
  sales, traffic, devices, countries, activity) tracks its own loading/error
  state independently, with its own retry button — one flaky endpoint never
  blanks the whole page.
- **Responsive**: sidebar becomes an off-canvas drawer under 760px; grids
  collapse from 4/3/2 columns down to 1 as the viewport narrows.
