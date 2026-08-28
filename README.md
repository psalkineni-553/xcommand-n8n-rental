# xCommand

xCommand Cloud is a short-term n8n workspace rental platform. It lets users launch an isolated n8n instance in the browser, complete checkout, and start building workflows without handling Docker, reverse proxies, SSL, or server setup themselves.

Live surfaces verified from the public site:

- `https://www.xcommand.cloud` - marketing site and plan overview
- `https://www.xcommand.cloud/ready.html` - workspace provisioning/status page
- `https://www.xcommand.cloud/support.html` - support and AI assistant page
- `https://app.xcommand.cloud` - app entrypoint used by the backend and frontend

## What The Product Does

- Launches private n8n workspaces on demand
- Supports free and paid rental plans
- Shows live free-plan capacity on the landing page
- Reuses the checkout email to locate workspaces
- Provides a support bot for workspace, billing, and expiry questions
- Cleans up expired workspaces automatically

## Public Site Experience

The public site presents xCommand Cloud as a fast way to get a ready-to-use n8n environment. The main customer-facing pages cover:

- Home page with hero, benefits, pricing, roadmap, team, testimonials, and CTA sections
- Ready page that shows provisioning status after payment
- Support page with FAQ and AI-assisted help

The current site messaging emphasizes:

- Isolated n8n containers
- Fast provisioning
- Temporary workspaces with expiry
- No setup burden for the user
- Simple plan choices for 24 hours or 5 days

## Repository Layout

```text
api/              FastAPI backend for provisioning, Stripe, support chat, and workspace lookup
web/              Production web app served at app.xcommand.cloud
frontend-lovable/  React + Vite source used to build the public site
worker/           Background worker processes
janitor/          Expiry cleanup and workspace removal jobs
migrations/       Database schema
monitoring/       Prometheus and Grafana config
traefik/          Reverse proxy config
scripts/          Local sync and publish helpers
landing/          Static landing site assets
infra/           Legacy and deployment-specific infrastructure layouts
```

## Architecture

The stack is built around Docker Compose and a set of cooperating services:

- `web` serves the customer-facing UI
- `api` handles workspace provisioning, Stripe checkout, the support bot, and workspace lifecycle endpoints
- `worker` handles background jobs
- `janitor` removes expired workspaces and cleans up resources
- `postgres` stores users, payments, and workspace records
- `grafana`, `prometheus`, `node-exporter`, and `cadvisor` provide observability

The backend provisions ephemeral n8n containers, labels them with expiry metadata, and stores the workspace record in Postgres. The frontend reads the free-plan capacity from the API and uses that to drive the landing page and pricing call to action.

## Tech Stack

- React
- TypeScript
- Vite
- Tailwind CSS
- shadcn/ui
- FastAPI
- PostgreSQL
- Docker
- Traefik
- Stripe
- OpenAI API for support chat

## Local Development

### Frontend

```bash
cd frontend-lovable
npm install
npm run dev
```

### Full stack

Use Docker Compose from the repository root after setting the required environment variables in `.env`.

```bash
docker compose up --build
```

## Environment

The root `.env.example` documents the expected runtime variables. The main groups are:

- PostgreSQL credentials
- Stripe secret and webhook secret
- Base domains for the app and workspace subdomains
- OpenAI configuration for the support assistant
- SMTP settings for notifications
- Docker host and encryption settings for workspace provisioning

## Key Backend Behavior

- `POST /stripe/create-checkout-session` creates Stripe Checkout sessions
- `POST /stripe/webhook` provisions a workspace after successful payment
- `POST /provision/free` creates or reuses a free workspace when capacity allows
- `GET /plans/free/status` exposes the live free-plan capacity
- `GET /workspaces/by-email/{email}` returns the latest workspace for an email
- `POST /support/chat` powers the AI support assistant

## Frontend Notes

The public UI is organized into sections such as:

- Hero
- About
- Features
- How It Works
- Roadmap
- Team
- Testimonials
- Pricing
- CTA

The home page also shows live free-workspace capacity and clear plan messaging, which keeps the marketing site aligned with backend state.

## Build And Sync Scripts

- `scripts/lovable.sh` updates the Lovable submodule, builds the frontend, and copies the generated site into `web/` and `infra/n8n/landing/`
- `scripts/push-frontend.sh` stages the generated frontend changes, creates a commit, and pushes to `origin/main`
- `gitpush-ui.sh` is a wrapper that runs the two scripts above and then prints the server-side deploy step

## Deployment Flow

The intended release flow is:

1. Sync and build the Lovable frontend
2. Commit the generated frontend and submodule pointer
3. Push to GitHub
4. SSH into the server
5. Run `./deploy.sh` in `/srv/xcommand-n8n-from-github`

## Operational Notes

- The repository uses a frontend submodule at `frontend-lovable`
- Workspace cleanup is time-based and handled outside the user-facing UI
- Support escalations should go through the support page and the documented email path
- Public-facing URLs should stay aligned with the live domains in the codebase

## Current Status

This repository contains both the product site and the infrastructure needed to run it. The codebase is structured for a production-style n8n rental service with billing, provisioning, support, and cleanup all represented in the repo.
