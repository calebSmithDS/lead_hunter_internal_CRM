# Lead Hunter (Internal CRM)

> Note: This README is for a proprietary internal WaaS (Website as a Service) application stored in a private repository.

## Overview

Lead Hunter is a custom lead generation and CRM tool that finds local businesses with missing, broken, or slow websites. It automates prospecting by scraping Google Maps, filtering out optimized competitors, and running Google PageSpeed audits on the targets to build actionable cold-calling lists.

<img width="2475" height="1279" alt="image" src="https://github.com/user-attachments/assets/301690ec-18de-4813-a50b-909073b2862b" />

## Architecture & Features

### Frontend

* **Tech:** Next.js (App Router), React 19, Tailwind CSS v4
* **Client-Side Processing:** The dashboard table handles complex sorting and filtering (by audit status, speed score, or website type) entirely in the browser using React's `useMemo` hook, keeping the interface fast without re-querying the database.
* **Optimistic UI:** Lead statuses (e.g., "Left Voicemail", "Meeting Booked") update instantly on the screen while synchronizing with the database in the background.

### API Layer

* **Tech:** Next.js Route Handlers, TypeScript
* **Asynchronous Triggers:** The `/api/jobs/trigger` route takes the user's matrix of locations and industries, dispatches the scraping job to Outscraper, and immediately returns a success response to free up the browser.
* **Webhook Ingestion:** Because large scrapes take minutes, a dedicated `/api/webhooks/outscraper` route listens for the returning JSON payload once the external job finishes.
* **Filter(Business logic):** The webhook evaluates the raw data on arrival. It discards businesses with strong custom domains and strictly retains leads with no website or just a Facebook page.
* **Audit Orchestration:** The `/api/jobs/audit` route calculates mobile load times (LCP), performance scores, and SEO metrics using the Google PageSpeed Insights API.

### Backend & Infrastructure

* **Tech:** Neon (Serverless Postgres), Prisma ORM, Upstash QStash, Vercel
* **Idempotent Storage:** Prisma's `upsert` function handles database writes. It uses Google's `place_id` as a unique identifier to prevent duplicate rows when searching overlapping territory combinations.
* **Background Queuing:** Leads that have a custom website are pushed into an Upstash QStash message queue. QStash feeds these URLs to the audit API one by one with a 15-second delay. This prevents Vercel serverless timeouts and bypasses Google's strict rate limits.
* **External Services:** Outscraper API (Google Maps data extraction) and Google PageSpeed Insights API (website performance metrics).
