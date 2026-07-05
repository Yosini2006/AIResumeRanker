# TalentLens (Single-File Edition)

A resume-vs-job-description ATS scoring tool, packaged as a single, dependency-free `index.html`. This is a client-side port of the original Next.js/Prisma "TalentLens" app — same scoring engine, same visual design, no server or database required.

## Quick start

Just open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). That's it — no build step, no `npm install`, no server.

You need an internet connection the first time you open it, since it loads one external script:
- **mammoth.js** (from cdnjs) — used to extract text from uploaded `.docx` files.

Everything else (styling, fonts, app logic, charts) is self-contained in the file.

## What it does

- **Landing page** — marketing page describing the product (features, pricing, FAQ).
- **Local "account"** — enter a name + email to create a session. No password, no real backend — this just labels your local data.
- **Analyze Resume** — paste or upload (`.txt` / `.docx`) a resume and a job description, then run a scan.
- **ATS scoring engine** — a rule-based (non-LLM) heuristic engine that compares the two texts and produces:
  - Overall ATS Score, Job Match Score, Keyword Match %
  - Technical / soft-skill / certification match breakdowns
  - Missing-skills table with priority (Required vs Preferred)
  - Resume formatting checks: bullet usage, action verbs, quantified achievements, passive voice, grammar/readability heuristics
  - A skill radar chart, a keyword composition donut chart, and category score bars
  - Prioritized, plain-English suggestions for improving the resume
- **History** — save scan results and browse them later.
- **Settings** — view your local account details.

## How data is stored

This build has **no server and no database**. Everything is kept in your browser's `localStorage`:

| Key            | Contents                                   |
|----------------|---------------------------------------------|
| `tl_user`      | `{ name, email }` for the local session     |
| `tl_reports`   | Array of saved scan reports                 |

Implications:
- Your data is private to this browser/device — it is never sent anywhere.
- Clearing your browser's site data (or using a different browser/device) will lose your saved reports.
- There is no real authentication — "logging in" just labels whatever you type as the current local user.

## File formats supported

| Format | Support |
|--------|---------|
| `.txt` | ✅ Read directly |
| `.docx`| ✅ Parsed via mammoth.js |
| `.pdf` | ❌ Not implemented — paste the text instead |

## Scoring methodology

The engine is keyword + heuristic based (not an LLM call), matching resume and job-description text against curated lists of:
- Technical skills (languages, frameworks, cloud, databases, etc.)
- Soft skills (communication, leadership, teamwork, etc.)
- Certifications (AWS, PMP, Scrum Master, etc.)

It also inspects structural signals: word count, bullet-point usage, action-verb density, quantified results (%, $, numbers), passive-voice usage, and estimated years of experience vs. what the job description requires. These are combined into weighted scores (see the `analyzeResume` function in `index.html` for exact weights).

Because it's rule-based, it's fast and fully transparent, but it won't catch semantic matches (e.g. "led a team" vs. "leadership") the way an LLM-based reviewer would. The `analyzeResume` function is a natural place to swap in an LLM API call for deeper analysis.

## Differences from the original Next.js app

The original was a full-stack app (Next.js + Prisma + NextAuth + Postgres). To make it a single portable HTML file, the following were adapted:

| Original | This version |
|---|---|
| NextAuth (email/OAuth login) | Local, unauthenticated "session" stored in `localStorage` |
| Prisma + Postgres (`ResumeReport` table) | Reports saved to `localStorage` |
| Recharts (React charting library) | Hand-rolled SVG charts (radar, donut, bars) |
| Tailwind (compiled at build time) | Tailwind CLI-compiled CSS, inlined into the file |
| Server-rendered pages / routing | Client-side view switching (vanilla JS) |

The scoring logic itself (`lib/atsEngine.ts` in the original) was ported line-for-line into JavaScript with no behavior changes.

## Customizing

Everything lives in one file, organized into clearly labeled sections:
- `<style>` — compiled Tailwind CSS + custom component styles
- ATS scoring engine — skill lists, weights, `analyzeResume()`
- Chart helpers — `ringSVG`, `radarSVG`, `pieSVG`, `bar100`
- Page renderers — `renderOverview()`, `renderAnalyze()`, `renderHistory()`, `renderSettings()`
- Landing page content — `FEATURES`, `PRICING`, `FAQS`, `TESTIMONIALS` arrays

To change the skill lists the engine matches against, edit `TECH_SKILLS`, `SOFT_SKILLS`, and `CERTIFICATIONS` near the top of the script.

## Known limitations

- No real backend, so there's no multi-device sync, team features, or billing.
- PDF resumes/job descriptions must be pasted as text.
- Scoring is heuristic, not semantic — wording matters (e.g. "Node.js" must appear as written to be detected).
