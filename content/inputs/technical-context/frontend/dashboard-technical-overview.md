---
title: "Dashboard Technical Overview"
publish: true
---

# Pinto Web App – Project Summary

## 📌 Project Overview

| Attribute | Details |
|-----------|---------|
| **Name** | `pinto-web-app` |
| **Purpose** | A modern **financial management dashboard** providing overview metrics, invoice/bill management, reporting, and business insights |
| **Target Audience** | Financial managers, accountants, and business administrators |
| **Framework** | Next.js 14 (App Router) with **static export** mode |
| **Private** | Yes |

### Core Features
- **Dashboard Overview** – High-level financial summary with top-three items and revenue vs. expenses charts
- **Inquiries** – Business inquiries tracking
- **Insights** – Business intelligence and analytics
- **Pending Bills** – Bill management and review
- **Pending Invoices** – Invoice management and review
- **Reports** – Financial report generation and viewing
- **Authentication** – Login flow with OTP/verification
- **WhatsApp Integration** – Direct WhatsApp chat support
- **PDF Viewing** – Document preview for invoices/bills

---

## 🏗 Architecture & Structure

```
src/
├── app/                          # Next.js App Router (routes & layouts)
│   ├── (auth)/                   # Auth route group (login, verification)
│   ├── (dashboard)/              # Dashboard route group (protected)
│   │   ├── dashboard/
│   │   │   ├── inquiries/
│   │   │   ├── insights/
│   │   │   ├── pending-bills/
│   │   │   ├── pending-invoices/
│   │   │   ├── reports/
│   │   │   └── page.tsx          # Dashboard overview
│   │   └── layout.tsx            # Dashboard layout with sidebar/nav
│   ├── (main)/                   # Main/public route group
│   ├── layout.tsx                # Root layout (font, AppWrapper)
│   └── not-found.tsx             # 404 page
│
├── features/                     # Feature-based modules
│   ├── dashboard/                # Dashboard feature
│   │   ├── columns/              # Table column definitions
│   │   ├── components/           # Dashboard-specific components
│   │   ├── hooks/                # Dashboard-specific hooks
│   │   ├── screens/              # Page-level screen components
│   │   ├── services/             # API services & data layer
│   │   │   ├── _api/             # React Query hooks
│   │   │   └── types.ts
│   │   └── store/                # Feature-level state (Zustand)
│   ├── home/                     # Home/landing page
│   ├── login/                    # Authentication feature
│   └── tasks-management/         # (Excluded from build – WIP)
│
├── shared/                       # Shared/reusable code
│   ├── component/                # Generic components (forms, loaders, etc.)
│   ├── hooks/                    # Global hooks (filter params)
│   ├── modal/                    # Confirmation drawer, modals
│   ├── store/                    # Global stores (confirm modal)
│   └── utilities/                # Utility functions (date, phone, currency)
│
├── libs/                         # External integrations & setup
│   ├── api.ts                    # SDK API factory
│   ├── axios-instance.ts         # Axios HTTP client setup
│   ├── query-provider.tsx        # TanStack Query provider
│   ├── tanstack-helpers.ts       # Query/mutation helper creators
│   ├── types.ts                  # Shared types
│   └── zustand.ts                # Zustand storage configuration
│
├── ui/                           # UI system
│   ├── helpers/                  # UI helpers
│   ├── theme/                    # MUI theme configuration
│   │   ├── components/           # Component theme overrides
│   │   ├── configs/              # Theme presets
│   │   ├── palette.ts            # Color palette
│   │   ├── typography.ts         # Typography scale
│   │   └── ...
│   └── toasts/                   # Toast notification system
│
└── config/
    └── api.ts                    # API URL configuration
```

### Architecture Pattern
- **Feature-Sliced Design (FSD-inspired)**: Features are isolated in `src/features/` with their own components, hooks, services, and stores
- **Shared Layer**: Cross-cutting concerns live in `src/shared/`
- **Route Groups**: Next.js route groups `(auth)`, `(dashboard)`, `(main)` for layout isolation

---

## 📦 Libraries & Packages

### Core Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `next` | 14.2.30 | React framework (App Router, static export) |
| `react` / `react-dom` | ^18 | UI library |
| `@mui/material` | ^6.1.8 | Material UI component library |
| `@mui/icons-material` | ^6.1.7 | Material UI icons |
| `@mui/x-date-pickers` | ^7.22.2 | Date/time pickers |
| `@emotion/react` + `@emotion/styled` | ^11.13.0 | CSS-in-JS styling for MUI |
| `@tanstack/react-query` | ^5.83.0 | Server state & data fetching |
| `@tanstack/react-query-devtools` | ^5.51.15 | Query debugging tools |
| `axios` | ^1.7.2 | HTTP client |
| `react-hook-form` | ^7.60.0 | Form management |
| `@hookform/resolvers` | ^5.1.1 | Form validation resolvers |
| `yup` | ^1.6.1 | Schema validation |
| `recharts` | ^3.6.0 | Charting/visualizations |
| `zustand` (via `@alpha.apps/react-common`) | – | Lightweight state management |
| `react-toastify` | ^10.0.5 | Toast notifications |

### PDF & Documents

| Package | Version | Purpose |
|---------|---------|---------|
| `@react-pdf-viewer/core` | ^3.12.0 | PDF rendering |
| `@react-pdf-viewer/default-layout` | ^3.12.0 | Default PDF layout |
| `@react-pdf-viewer/zoom` | ^3.12.0 | PDF zoom controls |
| `pdfjs-dist` | 3.11.174 | PDF.js distribution |

### Utilities

| Package | Version | Purpose |
|---------|---------|---------|
| `lodash` | ^4.17.21 | Utility functions |
| `date-fns` | 2.22.1 | Date manipulation |
| `libphonenumber-js` | ^1.12.9 | Phone number parsing/validation |
| `material-ui-phone-number` | ^3.0.0 | Phone input component |
| `uuid` | ^11.1.0 | UUID generation |
| `immer` | ^11.1.3 | Immutable state updates |
| `stylis-plugin-rtl` | ^2.1.1 | RTL support for styling |
| `react-intersection-observer` | ^9.16.0 | Intersection Observer hook |
| `react-multi-date-picker` | ^4.5.2 | Additional date picker |

### Internal/Custom Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `@alpha.apps/pinto-web-app-sdk` | 1.2.9 | Auto-generated API SDK/client |
| `@alpha.apps/react-common` | ^1.2.11 | Shared utilities (storage, etc.) |

### Dev Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `typescript` | 5.8.3 | Type checking |
| `eslint` | ^8 | Linting |
| `prettier` | ^3.3.2 | Code formatting |
| `@typescript-eslint/*` | ^7.14.1 | TypeScript ESLint rules |

---

## 🔑 Key Files & Functions

### Application Entry Points

| File | Description |
|------|-------------|
| `src/app/layout.tsx` | Root layout; applies `AppWrapper`, fonts (Figtree), and metadata |
| `src/app/(dashboard)/layout.tsx` | Dashboard layout with sidebar, mobile header, navbar; wrapped in `WithAuth` HOC |
| `src/shared/component/wrappers/app-wrapper.tsx` | Global provider wrapper (Theme, Query, Toasts, ConfirmationDrawer) |

### API & Data Layer

| File | Description |
|------|-------------|
| `src/libs/api.ts` | Exports `webAppApi` – SDK factory for all API endpoints |
| `src/libs/axios-instance.ts` | Configures Axios instance with interceptors (auth tokens, refresh) |
| `src/libs/tanstack-helpers.ts` | `createQuery` / `createMutation` helper factories |
| `src/features/dashboard/services/_api/queries.tsx` | React Query hooks: `useGetDashboardQuery`, `useGetInvoicesSummaryQuery`, etc. |
| `src/features/dashboard/services/_api/mutations.tsx` | Mutation hooks: `useGetInvoiceFileMutation`, `useGetBillFileMutation` |

### State Management

| File | Description |
|------|-------------|
| `src/libs/zustand.ts` | Configures `storage` utility for persisting user, tokens, active tab |
| `src/shared/store/confirm-modal-store.ts` | Zustand store for confirmation modal state |

### Theme & UI

| File | Description |
|------|-------------|
| `src/ui/theme/index.tsx` | MUI `ThemeProvider` with date-fns localization |
| `src/ui/theme/palette.ts` | Color system definition |
| `src/ui/theme/typography.ts` | Typography scale |

### Utilities

| File | Description |
|------|-------------|
| `src/shared/utilities/index.ts` | Phone formatting, time formatting, UTC date helpers, currency formatting, WhatsApp link generator |

### Shared Components

| File | Description |
|------|-------------|
| `src/shared/component/header.tsx` | Application header |
| `src/shared/component/footer.tsx` | Application footer |
| `src/shared/component/logo.tsx` | Brand logo component |
| `src/shared/component/generic-*.tsx` | Reusable form components (autocomplete, date picker, text field) |
| `src/shared/component/document-preview-dialog.tsx` | PDF/document preview dialog |

---

## ⚙️ Configuration & Environment

### Environment Variables

| Variable | Description |
|----------|-------------|
| `NEXT_APP_API_URL` | Backend API base URL (required) |
| `NEXT_APP_WHATS_APP_NUMBER` | WhatsApp contact number for support |
| `NEXT_APP_VERSION` | App version (hardcoded to `1.0.5` in `next.config.mjs`) |

### Build Configuration (`next.config.mjs`)

```js
output: 'export'        // Static site generation (SSG)
trailingSlash: true     // Adds trailing slashes to routes
images.unoptimized: true // No Next.js image optimization (static export)
webpack.alias.canvas = false // Excludes 'canvas' (PDF.js compatibility)
```

### TypeScript (`tsconfig.json`)

- **Path aliases**: `@/*` → `./src/*`
- **Strict mode**: Disabled (`strict: false`)
- **Excluded**: `tasks-management` feature directories (work-in-progress)

### Code Quality

- **ESLint**: Custom config with React, Prettier, and unused-imports plugins
- **Prettier**: Auto-formatting enabled (`prettier --write .`)

### NPM Registry (`.npmrc`)
- Likely configured for private/scoped `@alpha.apps` packages

---

## 🚀 Usage & Workflow

### Quick Start

```bash
# 1. Install dependencies
npm install   # or yarn

# 2. Set up environment
# Create .env file:
# NEXT_APP_API_URL=https://your-api-url.com
# NEXT_APP_WHATS_APP_NUMBER=1234567890

# 3. Run development server
npm run dev
# → http://localhost:3000

# 4. Build for production (static export)
npm run build
# → Output in /out directory

# 5. Serve production build
npm run start-export
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Build for production (static export to `/out`) |
| `npm run start` | Next.js start (not used with static export) |
| `npm run start-export` | Serve static `/out` directory |
| `npm run lint` | Run ESLint |
| `npm run format` | Format code with Prettier |

### Routing Structure

| Route | Description |
|-------|-------------|
| `/` | Home page |
| `/login` | Login page |
| `/login/verification` | OTP verification |
| `/dashboard` | Dashboard overview |
| `/dashboard/inquiries` | Inquiries page |
| `/dashboard/insights` | Business insights |
| `/dashboard/pending-bills` | Pending bills management |
| `/dashboard/pending-invoices` | Pending invoices management |
| `/dashboard/reports` | Financial reports |

---

## 📝 Potential Improvements & Notes

### ⚠️ Known Issues / Technical Debt

1. **Strict Mode Disabled** – `strict: false` in `tsconfig.json` may hide type safety issues. Consider enabling it gradually.
2. **Tasks Management Excluded** – The `tasks-management` feature is excluded from the build (likely incomplete/WIP).
3. **Hardcoded Version** – `NEXT_APP_VERSION` is hardcoded in `next.config.mjs` instead of pulled from `package.json`.
4. **No `.env` Template** – No `.env.example` file is provided for new developers.
5. **Commented-out Code** – Several files contain commented-out code (theme responsive fonts, encoding alias, ScrollTop component).


### 🏆 Strengths

- Clean **feature-sliced architecture** for maintainability
- Strong **type safety** with TypeScript
- Modern **React Query** data-fetching patterns
- Well-organized **MUI theme** system
- **Static export** for easy deployment (CDN-ready)
- Good use of **dynamic imports** to reduce initial bundle size
