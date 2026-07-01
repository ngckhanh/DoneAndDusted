# Done & Dusted

A Chrome extension that streamlines **Craft CMS → Jira handoffs**. It posts handoff comments to Jira issues with the correct template, status transition, and assignee — so QA and downstream teams get consistent, complete context without manual copy-paste.

---

## Overview

When content or site work is finished in Craft CMS, teams typically need to update the linked Jira ticket: add a structured handoff comment, move the issue to the right status, and assign it to the next owner. Done & Dusted automates that workflow from a browser side panel.

| What it does | How |
|---|---|
| Post handoff comments | Formats and sends comments to Jira via the REST API |
| Apply the right template | Uses predefined comment templates for consistent handoffs |
| Update issue status | Transitions the Jira issue to the correct workflow state |
| Set assignee | Assigns the ticket to the appropriate team member |

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | [React 19](https://react.dev/) |
| Build | [Vite 8](https://vite.dev/) + [@crxjs/vite-plugin](https://crxjs.dev/vite-plugin) |
| Styling | [Tailwind CSS 4](https://tailwindcss.com/) |
| Extension platform | Chrome Extension **Manifest V3** |
| Icons | [Lucide React](https://lucide.dev/) |
| Linting | ESLint 9 (flat config) |

---

## Project Status

This repository is in **early setup**. The build tooling, extension manifest, and environment scaffolding are in place. Application source files referenced by the manifest (side panel UI, background service worker, icons) are **not yet implemented**.

Planned entry points (from `manifest.json`):

```
src/
├── background.js          # Service worker — extension lifecycle & messaging
└── sidepanel/
    └── index.html         # Side panel UI (React app entry)
icons/
└── icons.png              # Extension icon (16 / 48 / 128 px)
```

---

## Prerequisites

- **Node.js** 18+ (20+ recommended)
- **npm** 9+
- **Google Chrome** (or Chromium-based browser with side panel support)
- A Jira Cloud site on `*.atlassian.net` with API access

---

## Getting Started

### 1. Clone and install

```bash
git clone https://github.com/ngckhanh/DoneAndDusted.git
cd DoneAndDusted
npm install
```

### 2. Configure environment variables

Copy the example env file and fill in your Jira credentials:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `VITE_JIRA_HOST` | Your Jira Cloud host, e.g. `your-org.atlassian.net` or `https://your-org.atlassian.net/` |
| `VITE_JIRA_EMAIL` | Email address of the Jira account used for API calls |
| `VITE_JIRA_API_TOKEN` | [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens) for the account above |

> **Security note:** Vite inlines `VITE_*` variables into the extension bundle at build time. Anyone with access to the built `dist/` folder can read them. Use for local development and team defaults — not for secrets that must stay hidden in production.

Re-run `npm run build` or `npm run dev` after changing `.env`.

### 3. Run in development

```bash
npm run dev
```

Vite watches for changes and rebuilds the extension. Load the output directory as an unpacked extension in Chrome (see below).

### 4. Load the extension in Chrome

1. Open `chrome://extensions`
2. Enable **Developer mode** (top right)
3. Click **Load unpacked**
4. Select the project's `dist/` folder (created after the first build)

The extension action opens the **side panel** defined in the manifest.

---

## Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start Vite dev server with hot reload for the extension |
| `npm run build` | Production build → outputs to `dist/` |
| `npm run preview` | Preview the Vite build (web preview; limited for extensions) |
| `npm run lint` | Run ESLint across the project |

---

## Extension Permissions

Defined in `manifest.json`:

| Permission | Purpose |
|---|---|
| `storage` | Persist user preferences and Jira configuration |
| `tabs` | Read context from the active browser tab (e.g. Craft CMS or Jira pages) |
| `windows` | Window-level extension behavior |
| `clipboardRead` | Read clipboard content for handoff workflows |
| `sidePanel` | Render the main UI in Chrome's side panel |
| `https://*.atlassian.net/*` | Call the Jira Cloud REST API |

---

## Architecture (Planned)

```
┌─────────────────────────────────────────────────────────┐
│                     Chrome Browser                       │
│                                                          │
│  ┌──────────────┐    messages     ┌──────────────────┐  │
│  │  Side Panel  │ ◄──────────────►│  Background SW   │  │
│  │  (React UI)  │                   │  (background.js)│  │
│  └──────┬───────┘                   └────────┬─────────┘  │
│         │                                    │            │
│         │         Jira REST API              │            │
│         └────────────────┬───────────────────┘            │
│                          ▼                                │
│               https://*.atlassian.net                     │
└─────────────────────────────────────────────────────────┘
```

**Side panel** — React app where users compose or confirm handoff details (template, status, assignee) and trigger the post.

**Background service worker** — Handles extension lifecycle, cross-context messaging, and API calls that should not run in the UI thread.

**Jira integration** — Authenticated requests to Jira Cloud using Basic auth (`email` + `api token`) against the configured host.

---

## Configuration Files

| File | Role |
|---|---|
| `manifest.json` | Chrome extension manifest (MV3) — permissions, entry points, metadata |
| `vite.config.js` | Vite + React + Tailwind + CRXJS plugin wiring |
| `eslint.config.js` | ESLint flat config with React Hooks and Chrome globals |
| `jsconfig.json` | JS/JSX editor support (ES2022, React JSX) |
| `.env.example` | Template for required Jira environment variables |

---

## Jira API Token Setup

1. Sign in at [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click **Create API token** and copy the value
3. Set `VITE_JIRA_EMAIL` to the account email and `VITE_JIRA_API_TOKEN` to the token
4. Set `VITE_JIRA_HOST` to your site hostname (e.g. `acme.atlassian.net`)

---

## Contributing

1. Create a feature branch from `main`
2. Make changes and run `npm run lint`
3. Test by loading the unpacked extension from `dist/`
4. Open a pull request with a clear description of the workflow change

---

## License

Not specified yet. Add a `LICENSE` file when the project is ready for wider distribution.

---

## Related Links

- Repository: [github.com/ngckhanh/DoneAndDusted](https://github.com/ngckhanh/DoneAndDusted)
- [CRXJS Vite Plugin docs](https://crxjs.dev/vite-plugin)
- [Chrome Side Panel API](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)
- [Jira Cloud REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)
