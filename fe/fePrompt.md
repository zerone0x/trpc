Build a Next.js 14 (React 18) web client for a text-based MMORPG called “Berkeley World”.

Overall goals:
• First interactive paint ≤ 1.5 s on 4G
• WebSocket text-stream latency ≤ 300 ms
• Command-send / feedback error rate < 0.1 %

Tech stack:
– Next.js 14 App Router (RSC + CSR where noted)  
– TypeScript 5 strict  
– Tailwind CSS + shadcn/ui  
– Zustand for local UI state  
– TanStack Query for server data  
– WebSocket primary (fallback SSE)  
– NextAuth.js with email + OAuth (JWT)  
– Vite for dev build (or Turbopack later)  
– Next.js API routes for MVP; migrate to Fastify microservice when custom model or performance tuning is required  
– ESLint, Prettier, Vitest

Routes:
• “/” static marketing page  
• “/login” RSC (email/OAuth auth form)  
• “/play” RSC + CSR (main game UI, auth-required)  
• “/profile” RSC (user info)

Component breakdown:
GameLayout  
 Sidebar  
 – online-players list  
 – system menu  
 MainWindow  
 – MessageStream (append-only, time-ordered)  
 – CommandInput (↑↓ history, Tab autocomplete)  
 HUD (location, status, inventory summary)  
AuthForm  
Reusable primitives: UserAvatar, Tooltip, etc.

State & data flow:
React UI → Zustand (local) + TanStack Query (HTTP/WS to Fastify backend — or temporary Next.js API routes during MVP)  
WebSocket hooks: useGameStream, usePlayerPresence

Backend contract examples:
POST /api/action { text: string }  
WebSocket wss://domain.com/ws/play?token=…  
Stream frame: { type:“chunk”, content:“…” } (≤ 50 chars, ordered)  
Client reassembles by sequence, auto-scroll (toggleable), store last 500 messages in IndexedDB.

Performance & DX:
• next/font google subsets  
• Suspense + use streaming for progressive render  
• Tree-shaking and route-level code splitting  
• Images via next/image lazy

Accessibility & i18n:
• Complete tabIndex + aria attributes  
• Adjustable text direction and size  
• next-intl (default zh-CN, prepare en)

Testing:
• Unit: Vitest + @testing-library/react  
• Component: Playwright Component  
• E2E: Playwright (login → command → feedback loop)  
CI runs all tests on main.

Development workflow:
• Front-end first: stub/mocked fetch (e.g., MSW or static JSON) to unblock UI while backend is under development  
• Swap stubs with real Fastify/Next.js endpoints once ready  
• Keeps front/back teams decoupled and accelerates iteration

Directory sketch:
src/app/layout.tsx  
src/app/page.tsx  
src/app/play/… (page.tsx, GameLayout.tsx, MessageStream.tsx, CommandInput.tsx)  
src/components/ui/…  
src/hooks/useGameStream.ts, useZustandStore.ts  
src/lib/api.ts (fetch wrapper)  
src/lib/ws.ts (WebSocket client)

Iterative plan:
Sprint 1: route scaffold + Auth UI  
Sprint 2: WebSocket channel & stream render  
Sprint 3: HUD, Sidebar, command history  
Sprint 4: performance & a11y polish
