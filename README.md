# Software Engineering at Scale

A 5-day intern course covering the systems, tradeoffs, and practices that separate production-grade software engineering from tutorial-level coding. Built and taught by **Aashutosh**.

Live deck: https://aashu10sh.github.io/seas26/

---

## About the Course

This course was created in response to a recurring problem: interns writing code that works, but that they don't actually understand — code with no model of what happens once it hits real traffic, real failure modes, or real scale.

Rather than a standard frontend/backend/database walkthrough, this course goes deep into the concepts that govern how production systems actually behave: compute models, database internals, distributed communication, and deployment infrastructure. Each session is one hour, runs Monday through Friday, and closes with a short (15-minute) assignment designed to reinforce reasoning over rote recall.

**Format:** 5 sessions × 1 hour (Mon–Fri)
**Audience:** University interns (open beyond DMT interns — friends interested in the material are welcome to join)
**Prerequisites:** Basic programming proficiency. No prior distributed systems knowledge assumed.

---

## Syllabus

| Day | Topic | Focus |
|---|---|---|
| **1** | Software Engineering Components | Builds a system from a single problem statement — compute → DB → object store → queues → async processing → batching → CI/CD & tests — to motivate *why* each component exists, not just what it is. |
| **2** | Deep Dive into Compute | CPU-bound vs IO-bound work, CPU cores & true parallelism, threads, green threads/coroutines, parallelism vs concurrency, serverless, vertical vs horizontal scaling. |
| **3** | Deep Dive into Databases | OLTP vs OLAP, CAP theorem, B-tree vs LSM tree engines, replication, sharding. Case studies contrasting a consistency-paramount system with an eventually-consistent one. |
| **4** | Deep Dive into Service Communication | Sync vs async communication, RPC vs REST vs GraphQL, serialization & schema evolution, idempotency, retries, timeouts, backpressure, circuit breakers. |
| **5** | Deployment, Delivery & Architecture | Monolith vs microservices, deployment strategies (rolling/blue-green/canary), containers, Kubernetes, IaC, CI/CD pipelines, reverse proxies/ingress, observability, auto scaling. |

Each day builds on the last. The arc across the week is:
**what exists → how it computes → how it stores state → how it talks → how it ships and stays alive.**

### Assignments

Every session ends with a 15-minute assignment — deliberately short, scenario-based, and reasoning-focused rather than code-heavy, so they don't add fatigue on top of the session itself.

| Day | Assignment |
|---|---|
| 1 | **Component Spotting** — Identify which components a familiar app (e.g. Instagram) uses, and why. |
| 2 | **Bound or Not** — Classify given tasks as CPU-bound or IO-bound and justify the right compute model. |
| 3 | **Pick Your Tradeoff** — Given 3 scenarios, decide consistency vs availability under a network partition. |
| 4 | **Idempotency Bug Hunt** — Spot the missing idempotency key in a retry-on-failure snippet and fix it. |
| 5 | **Deploy or Don't** — Pick and justify a deployment strategy for a high-stakes hotfix scenario. |

---

## Tech Stack

This deck is built with [Slidev](https://sli.dev) — a developer-first presentation tool that lets slides be written in Markdown, with support for live code, diagrams, and embedded Vue components.

- **Slidev** — core presentation framework
- **Mermaid** — used throughout for architecture, sequence, and flow diagrams (system diagrams, CAP theorem partition scenarios, B-tree/LSM tree structure, replication flow, CI/CD pipelines, Kubernetes cluster layout, etc.)
- **Vue components** — used for interactive, hands-on explainers where a static diagram isn't enough (e.g. a CAP theorem partition toggle, an idempotent-vs-non-idempotent retry simulator, a distributed trace waterfall visualization)
- **GitHub Pages** — hosting/deployment for the public deck

---

## Project Structure

```
.
├── slides.md              # Entry deck / index
├── days/
│   ├── day1.md             # Software Engineering Components
│   ├── day2.md             # Deep Dive into Compute
│   ├── day3.md             # Deep Dive into Databases
│   ├── day4.md             # Deep Dive into Service Communication
│   └── day5.md             # Deployment, Delivery & Architecture
├── components/             # Custom interactive Vue components used across days
├── public/                 # Static assets (images, etc.)
└── package.json
```

---

## Running Locally

```bash
# install dependencies
npm install

# start the dev server (with hot reload)
npm run dev
```

Then open the local URL Slidev prints (default `http://localhost:3030`).

### Useful Slidev commands

| Command | Purpose |
|---|---|
| `npm run dev` | Local dev server with hot reload |
| `npm run build` | Build the static deck for deployment |
| `npm run export` | Export the deck to PDF/PNG |

### Navigating the deck

- `→` / `Space` — next slide/step
- `←` — previous slide/step
- `f` — toggle fullscreen presenter mode
- `o` — overview mode (see all slides at once)

Full Slidev docs: https://sli.dev

---

## Deployment

The deck is deployed to GitHub Pages at https://aashu10sh.github.io/seas26/. Since this is a static, edge-cached deployment, changes may take a short time to propagate globally after a push — if the live URL looks stale immediately after deploying, that's expected CDN propagation lag, not a broken deploy.

---

## Contributing / Notes for Future Sessions

- Each day's content lives independently in `days/dayN.md` — days can be edited without affecting others.
- Interactive components are kept in `components/` and reused across days where the concept repeats (e.g. consistent visual language for architecture diagrams throughout the week).
- Diagrams are written natively in Mermaid inside the markdown rather than as external images, so they stay easy to edit as the course evolves.
- If extending this course beyond 5 days or reusing it for a different cohort, keep each day's opening "hook" scenario concrete and specific — abstract or trivia-style openers land noticeably worse than real, stakes-driven scenarios.

---

## Acknowledgements

Course designed and taught by Aashutosh, based on lessons learned building distributed systems in industry, with the goal of giving interns a real mental model of production software — not just syntax.
