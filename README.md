<div align="center">

# NeuroEase AI

### Your Daily Neuro Companion

An intelligent, **Arabic-first** (RTL) + English platform for tracking everyday neurological symptoms, learning each person's triggers, predicting the daily likelihood of an attack, and giving doctors structured, trustworthy patient data.

`Laravel 13` · `Flutter 3.44` · `OpenAI Responses API` · `SQLite / MySQL` · `Tailwind v4` · `mPDF`

**Live:** [https://spu.gnolexai.com](https://spu.gnolexai.com)

</div>

> ⚠️ **NeuroEase AI is a monitoring aid, not a diagnosis.** It never prescribes or changes medication. All medical wording, scoring bands and safety rules must be reviewed by a licensed neurologist before real-world use (see [`docs/CLINICAL_REVIEW.md`](docs/CLINICAL_REVIEW.md)).

---

## Table of contents

- [What is NeuroEase AI?](#what-is-neuroease-ai)
- [Conditions covered](#conditions-covered)
- [Key features](#key-features)
- [Architecture](#architecture)
- [The intelligence engine](#the-intelligence-engine)
- [Safety & privacy](#safety--privacy)
- [Repository structure](#repository-structure)
- [Getting started](#getting-started)
- [Production deployment](#production-deployment)
- [Testing & evaluation](#testing--evaluation)
- [Design system](#design-system)
- [Documentation](#documentation)
- [Team](#team)

---

## What is NeuroEase AI?

Migraine and related neurological conditions are among the world's leading causes of disability, yet day-to-day tracking still relies on memory and paper diaries. Good Arabic-language tools are rare, and neurologists usually see patients with no structured data from the weeks between visits.

NeuroEase AI solves this with three connected parts:

| Part | For | What it does |
| --- | --- | --- |
| 📱 **Patient mobile app** | Patients | 60-second daily check-in, one-tap attack logging, daily risk + micro-plan, personal insights, AI assistant, PDF report — works offline |
| 🖥️ **Doctor & admin web dashboard** | Neurologists, clinic coordinators, admins | Smart triage queue, full patient workspace, AI clinical summary, pre-visit briefing, alerts, population analytics, content & user management |
| 🧠 **Intelligence engine** | Both | Transparent analytics + prediction, plus a guarded language layer for explanations, chat and summaries |

It relies **only on what the user enters** — no wearables or medical devices required.

## Conditions covered

- **Migraine** (flagship) — with/without aura, full phase model: prodrome → aura → headache → postdrome
- Tension-type headache
- Chronic neurological fatigue
- Chronic stress with neurological expression
- Sleep disturbances
- Focus & mood fluctuations

## Key features

### Patient app (Flutter)

- **Onboarding** with separate, reversible consents, baseline profile, doctor link via invitation code, reminders and an honest *2–3 week learning period*.
- **Today screen** — risk card (level, confidence, explanation, top factors, migraine vs. nervous strain), three-action **micro-plan**, guilt-free streak, learning progress.
- **60-second check-in** — one question per card, big tap targets, auto-advance, haptics, smart defaults ("same as usual"), adaptive follow-ups on hard days, optional voice notes with AI extraction.
- **"I'm having an attack now"** — offers Low-Stimulation Mode, running timer, minimal taps, red-flag screening, soothing routine; close the attack later with peak, impact, relief and medication effect.
- **History** — timeline + gray-intensity calendar heat map, detailed attack pages.
- **Insights** — ranked personal triggers with evidence and confidence, trigger combinations, weekday/time patterns, trends, "what's working", medication-use days, validated questionnaire trends (MIDAS, HIT-6, sleep, stress, mood/focus).
- **Ask AI Neuro** — grounded chat in Arabic (incl. dialect) and English, suggested questions, voice input, share with doctor.
- **My doctor** — sharing toggle, doctor recommendations, appointments, one-tap **PDF report**.
- **Calm routines**, privacy center (export / delete), emergency card, trusted-person sharing, dark mode, text size, biometric unlock.
- **Offline-first** — check-ins and attacks are queued locally and synced automatically.

### Doctor & admin dashboard (Laravel)

- **First-launch setup wizard** — creates the root admin and clinic; optional realistic demo data.
- **Today's overview** — tiles, smart **priority queue** with plain-language reasons, recent alerts, today's visits with *Prepare for visit*.
- **Patient workspace (11 tabs)** — AI clinical summary with "show the data behind this", attacks, trends with treatment markers, triggers, medication & overuse, raw check-ins, notes & recommendations, treatment timeline with before/after, alerts, reports, consent & access log.
- **Pre-visit briefing** and print-friendly **PDF reports** (Arabic shaping, grayscale).
- **Smart alerts** — red flags, rising frequency, medication overuse, inactivity, baseline departure, missed appointments, overdue questionnaires, distress; acknowledge / snooze / assign / resolve; per-doctor thresholds.
- **Population analytics** with sample sizes and anonymized CSV export.
- **Education content** — bilingual, preview-as-patient, approval workflow.
- **Admin** — users & roles, clinics, invitations, audit & consent records, AI usage + engine evaluation, data requests. Idle auto sign-out and optional 2-step verification.

## Architecture

```
┌────────────────────┐   ┌──────────────────────────┐
│ Patient app        │   │ Doctor / Admin dashboard │
│ Flutter · offline  │   │ Blade · Alpine · Charts  │
└─────────┬──────────┘   └────────────┬─────────────┘
          │ HTTPS JSON (Sanctum)      │ HTTPS (session)
┌─────────▼───────────────────────────▼─────────────┐
│ Laravel 13 backend — https://spu.gnolexai.com     │
│  REST API · Web controllers · Intelligence Engine │
│  AI layer (guardrails) · Reports (mPDF)           │
└─────────┬─────────────────────────────┬───────────┘
          │                             │ de-identified JSON,
┌─────────▼─────────┐        ┌──────────▼───────────┐ structured outputs
│ SQLite / MySQL    │        │ OpenAI Responses API │
└───────────────────┘        └──────────────────────┘
```

## The intelligence engine

A **hybrid, transparent** design — the language model never produces risk numbers.

**Layer A — Personal analytics & prediction (deterministic)** · `web_Neuro/app/Services/Engine`

- **Personal baseline** for every daily factor; deviations are judged against the person, not a population.
- **~25 trigger features** per day (short sleep, 3-day stress build-up, late caffeine, caffeine drop, skipped meal, screens, cycle phase, weather…) including lagged effects.
- **Trigger analysis** — lift, log-odds and confidence vs. attacks within 24 h, plus trigger **combinations**.
- **Risk model** — `logit(p) = personal base rate + Σ weights × flags`, blending clinical priors with personal evidence; modes *prior → blended → personal* are always disclosed.
- **Micro-planner** that learns which actions actually help each person.
- Medication-overuse detection, red-flag detection (Arabic dialect + English), validated questionnaire scoring, alerting and triage.
- **Synthetic persona generator + evaluator** reporting AUC, Brier score and calibration.

**Layer B — Language intelligence (OpenAI)** · `web_Neuro/app/Services/AI`

| Task | Model |
| --- | --- |
| Assistant chat, clinical summary, pre-visit briefing | `gpt-6.1-sol` |
| Note extraction, explanation rewording | `gpt-6-luna` |
| Voice notes | `gpt-4o-transcribe` |

Strict JSON-schema structured outputs, grounding in the patient's own data, safety checks before and after every call, caching, per-patient limits, and full deterministic fallbacks — **everything works without an API key**. Details in [`docs/AI_DESIGN.md`](docs/AI_DESIGN.md).

## Safety & privacy

- 🚨 **Red flags** (sudden worst-ever headache, fever + stiff neck, one-sided weakness, speech loss, vision loss, seizure, fainting…) trigger a calm full-screen urgent-care message and a **critical alert** to the doctor (with consent).
- 💬 **Distress / self-harm** language gets a warm, supportive response with crisis guidance.
- 💊 Never diagnoses, never doses, never contradicts the treating doctor.
- 🔐 Separate, reversible consents; doctors see only what patients share; role-based access with audit log.
- 🕶️ Minimum, **de-identified** data sent to AI (`store: false`); API keys live only on the server.
- 📦 Patient data export and account deletion built in.

## Repository structure

```
Neruo_SPU/
├── web_Neuro/                 # Laravel backend + web dashboard
│   ├── app/Services/Engine/   # Layer A: risk, triggers, planner, alerts, evaluator…
│   ├── app/Services/AI/       # Layer B: OpenAI client, prompts, guardrails, assistant
│   ├── app/Services/Reports/  # PDF reports, data export
│   ├── app/Http/Controllers/  # Web/ (dashboard) and Api/ (mobile)
│   ├── resources/views/       # Blade UI
│   ├── lang/{ar,en}/          # All copy, Arabic + English
│   ├── config/neuroease.php   # Model routing, thresholds, engine parameters
│   └── tests/Feature/         # Patient journey, engine, AI guardrails
├── mob_Neuro/                 # Flutter patient app
│   └── lib/{core,state,widgets,features}/
└── docs/
    ├── AI_DESIGN.md
    ├── CLINICAL_REVIEW.md
    └── report/                # Graduation report generator (.docx)
```

## Getting started

### Requirements

PHP 8.3+, Composer, Node.js 20+, Flutter 3.44+ (Dart 3.12), and optionally MySQL.

### 1. Backend & dashboard

```bash
cd web_Neuro
composer install
npm install && npm run build
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve            # http://127.0.0.1:8000
```

Open the URL → the **setup wizard** creates your root admin. Tick *Load demo data* to explore with realistic patients.

<details>
<summary><b>Demo accounts</b> (when demo data is loaded — password <code>Demo@12345</code>)</summary>

| Role | Email |
| --- | --- |
| Doctor | `doctor@demo.neuroease.local` |
| Doctor 2 | `doctor2@demo.neuroease.local` |
| Coordinator | `coordinator@demo.neuroease.local` |
| Patients (app) | `layla@`, `omar@`, `sara@`, `khalid@`, `nour@`, `rami@demo.neuroease.local` |

Invitation codes: `DEMO-LINA`, `DEMO-OMAR`.
</details>

**Enable AI (optional)** — add to `web_Neuro/.env`:

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL_REASONING=gpt-6.1-sol
OPENAI_MODEL_FAST=gpt-6-luna
OPENAI_MODEL_TRANSCRIBE=gpt-4o-transcribe
```

### 2. Mobile app

```bash
cd mob_Neuro
flutter pub get
flutter run                                   # connects to https://spu.gnolexai.com/api
flutter run --dart-define=API_BASE=http://10.0.2.2:8000/api   # local backend (Android emulator)
flutter build apk --release
```

The server address can also be changed from **Server address** on the sign-in screen.

## Production deployment

1. Point the sub-domain document root to `web_Neuro/public`.
2. `composer install --no-dev --optimize-autoloader && npm ci && npm run build`
3. `cp .env.production.example .env` → set DB, mail and `OPENAI_API_KEY`, then `php artisan key:generate`.
4. `php artisan migrate --force && php artisan storage:link && php artisan optimize`
5. Open the site and complete the setup wizard.

Trusted proxies, forced HTTPS, secure cookies and restricted CORS are already configured.

## Testing & evaluation

```bash
cd web_Neuro && php artisan test
```

| Check | Result |
| --- | --- |
| Automated tests | **37 passing** (205 assertions) — full patient journey, setup wizard, role isolation, engine, AI guardrails |
| Red-flag detection | **20 / 20** test phrases (Arabic dialect + English), 0 / 5 false positives |
| Engine on synthetic personas | **AUC 0.657**, positive Brier skill, calibrated mid-range probabilities |
| Flutter static analysis | No errors or warnings |

> Accuracy is measured on synthetic data with known ground truth; real-world accuracy requires validation with consented users.

## Design system

- **Neutral palette only** — warm off-white "paper", near-black ink, grays. Severity is shown by weight, shape, filled segments and labels, never color.
- **Dark mode** and **Low-Stimulation Mode** (reduced contrast, no animation) for light-sensitive days.
- Arabic-first RTL with full English mirroring, large touch targets, scalable text, calm tone of voice.

## Documentation

| Document | Contents |
| --- | --- |
| [`docs/AI_DESIGN.md`](docs/AI_DESIGN.md) | Engine design, model selection, prompts, guardrails, privacy, evaluation |
| [`docs/CLINICAL_REVIEW.md`](docs/CLINICAL_REVIEW.md) | Clinical content review checklist |
| [`web_Neuro/AGENTS.md`](web_Neuro/AGENTS.md) | Developer conventions & verification commands |
| `NeuroEase_AI_Documentation.docx` | Full graduation report (Arabic) |

## Team

**Syrian Private University (SPU)** — Faculty of Informatics Engineering, Department of Software Engineering and Information Systems · 2026/2027

| | |
| --- | --- |
| **Students** | Mohammad Omar Mohammad Bassam Al-Set · Madyan Abdulaziz Al-Awad · Haidara Ali Al-Taweel |
| **Supervisor** | Dr. Ikram Massouh |

---

<div align="center">
<sub>NeuroEase AI — a monitoring aid, not a diagnosis. Built with care for people living with migraine.</sub>
</div>
