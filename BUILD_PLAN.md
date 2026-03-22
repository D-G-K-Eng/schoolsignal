# Sidekick — Build Plan
*Last updated: 2026-03-22*

---

## Guiding Principle

Build the smallest thing that is genuinely useful to Doug and Grayson first. Prove the model works. Then open to other parents.

---

## Phase 1 — Core (Target: 4 weeks)

### Week 1 — Foundation
**Goal:** Backend running, Gmail connected, emails flowing in

- [ ] Project scaffolding: FastAPI backend, React+Vite frontend, PostgreSQL on Railway
- [ ] GitHub repo, local dev environment, Railway + Vercel deployments wired up
- [ ] User auth (Supabase — email/password, no OAuth complexity yet)
- [ ] Gmail OAuth integration (Google Cloud project, OAuth consent screen, token storage)
- [ ] Email sync job: pull emails from connected Gmail inbox, store raw in DB
- [ ] Child + school domain setup (user specifies which domains = school)

**Done when:** Doug can connect his Gmail and see raw school emails appearing in the database.

---

### Week 2 — Intelligence
**Goal:** AI classification pipeline working end-to-end

- [ ] Classification pipeline: send each new school email to Claude Haiku
- [ ] Structured output: classification, title, due_date, child, action, confidence
- [ ] Signal storage: classified results saved as Signals in DB
- [ ] Handle Gmail forwards correctly (Classroom notifications forwarded from Grayson's account)
- [ ] De-duplication: don't classify the same email twice
- [ ] Basic error handling + retry logic

**Done when:** New school emails arrive, get classified, and signals appear in the DB with correct labels.

---

### Week 3 — UI + Calendar
**Goal:** Parent-facing app that's actually usable

- [ ] Signal feed: list view of action_required and reminder items, sorted by urgency/due date
- [ ] Status actions: mark done, snooze, dismiss
- [ ] Filter by child
- [ ] Full email view on click
- [ ] Calendar integration: Google Calendar events created for items with due dates
- [ ] Homework links attached to calendar events (extracted from email body)
- [ ] Morning digest: daily email summary (9am, configurable)

**Done when:** Doug opens the app on his phone, sees Grayson's school signals, taps an item, sees the full email and a calendar event has been created with the homework link.

---

### Week 4 — Polish + Beta
**Goal:** Stable enough for 5-10 beta users

- [ ] Multi-child support (tag signals by child, filter feed)
- [ ] Onboarding flow (connect Gmail, set school domains, add children)
- [ ] PWA config (installable on iPhone home screen, works offline for cached content)
- [ ] Basic settings (digest time, notification preferences)
- [ ] Manual "sync now" button
- [ ] Bug fixes from Doug's daily use
- [ ] Invite 5-10 beta parents (Jason, Ben, friends)

**Done when:** 5+ parents are using it daily without hand-holding.

---

## Phase 2 — Growth (Month 2)

- [ ] Push notifications (Firebase Cloud Messaging)
- [ ] Outlook + Microsoft 365 support
- [ ] iCloud Mail support (IMAP)
- [ ] Shared access: two parents on one account
- [ ] PDF/attachment handling: permission slips surfaced prominently
- [ ] Weekly child summary report
- [ ] Stripe subscription billing ($14.99/month)
- [ ] Paywall: free tier (1 child, Gmail, digest only) vs. paid

---

## Phase 3 — Scale (Month 3+)

- [ ] SMS delivery via Twilio
- [ ] WhatsApp delivery
- [ ] Apple Calendar support
- [ ] Multilingual classification (French, Spanish)
- [ ] School / district portal (structured push, no email scraping)
- [ ] Teacher-side tools
- [ ] SOC2 compliance groundwork (needed for district deals)

---

## Tech Stack Decisions

| Layer | Choice | Why |
|---|---|---|
| Backend | FastAPI (Python) | Best AI library support, fast to build |
| Frontend | React + Vite | Familiar, fast, PWA-ready |
| Database | PostgreSQL | Railway, managed, scales fine |
| Auth | Supabase | Don't build auth ourselves |
| Email (Gmail) | Google Gmail API | OAuth, reliable, official |
| Email (Outlook) | Microsoft Graph API | Phase 2 |
| AI | Claude Haiku | Fast, cheap, accurate for classification |
| Calendar | Google Calendar API | Phase 1; Apple Calendar Phase 3 |
| Push | Firebase Cloud Messaging | Phase 2 |
| SMS | Twilio | Phase 2/3 |
| Hosting | Vercel (frontend) + Railway (backend+DB) | Already know it, proven on Spectral |
| Billing | Stripe | Standard |

---

## Critical Path Items (Don't Skip)

1. **Google OAuth app verification** — start the Google Cloud project and OAuth consent screen on Day 1. Google reviews apps that request Gmail access. Can take days to weeks. Don't let this block the launch.

2. **Classification accuracy** — test with real Grayson emails before beta. One missed `action_required` (field trip permission, payment due) destroys trust. Tune the prompt until it's >95% accurate on real data.

3. **Gmail forward handling** — Grayson's Google Classroom notifs forward to Doug's inbox. The classifier needs to correctly attribute these to Grayson, not treat them as Doug's own school emails. Test this specifically.

4. **Don't break the existing setup** — `gmail_checker.py` keeps running independently. Sidekick uses OAuth, not IMAP. Zero risk of interference, but verify before going live.

---

## Definition of "Done" for Phase 1

- Doug uses Sidekick as his primary way to track Grayson's school stuff
- Zero missed action_required items over 2 weeks of real use
- At least 5 beta parents onboarded and using it
- Calendar events with homework links working
- Morning digest landing in inbox daily
