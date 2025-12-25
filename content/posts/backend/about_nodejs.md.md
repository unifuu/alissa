---
title: About Node.js
date: 2025-12-22
summary: 📝 Quick notes on Node.js...
tags:
  - backend
  - javascript
---

## Definition
- A JavaScript runtime built on Chrome's V8 engine that executes JS outside the browser.
- It is not a language or a framework; it is an environment that enables server-side JavaScript.
## Components
- Includes a set of built-in modules (File System, HTTP, Path) to handle system-level tasks.
## Benefits
- Unified language (JS / TS)
	- Faster onboarding
	- Shared types (DTOs, schemas)
	- Easier full-stack development
- Excellent for I/O-heavy workloads
	- High concurrency
	- Many fast requests
	- Real-time connections
## Applications
- Backend APIs:
	- REST APIs
	- GraphQL APIs
	- Authentication (JWT, OAuth)
	- Business logic
	- Connect to DB / Redis / Message Queue
```
Client → HTTP → Node.js → Database
```
- Real-time Systems
	- Event-driven
	- Non-blocking (async I/O)
	- Single-threaded but highly concurrent
- BFF (Backend for Frontend)
	- Aggregates data from multiple services (microservice architectures)
	- Shapes data specifically for frontend needs
- Frontend Tooling
	- Vite
	- Webpack
	- ESLint
	- NPM / YARN
- Full-Stack / SSR (Next.js, Nuxt)
	- Next.js (React)
	- Nuxt (Vue)