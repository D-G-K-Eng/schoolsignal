# SchoolSignal — Product Spec
*Last updated: 2026-03-22 | Built by Doug + Hammerz*

---

## The Problem

Parents receive an overwhelming volume of school communications — emails, newsletters, permission slips, reminders, event notices — across multiple channels. Nothing is prioritized. Important deadlines get buried. Busy parents miss things that matter.

No one has built a clean, opinionated filter for this.

---

## The Solution

SchoolSignal connects to a parent's email, reads incoming school communications, classifies them by urgency and type, and surfaces only what requires attention — at the right time, in the right format.

**Core promise:** You never miss something important from your kid's school again.

---

## Who Uses It

**Primary user:** Parents of school-age children (K-12)
- Busy, time-constrained
- Receive 5-20 school emails per week
- Currently managing this with inbox + mental notes + missed things
- Not necessarily technical

**Secondary user (future):** Schools / districts pushing structured data directly into the platform

**Non-user:** Teachers, administrators (they are the source, not the consumer)

---

## Core Feature Set

### Phase 1 — MVP (Month 1-2)

**Email connection**
- Connect Gmail via OAuth (Google API)
- Connect Outlook / Microsoft 365 via OAuth
- iCloud Mail via IMAP (app password)
- User specifies which senders / domains are "school" (e.g. @lbpsb.qc.ca)

**AI classification**
- Classify every incoming school email into:
  - `action_required` — permission slip, payment due, form to sign, RSVP
  - `reminder` — upcoming event, deadline approaching
  - `fyi` — newsletter, general update, no action needed
  - `ignore` — unsubscribe-worthy noise
- Extract: due date (if any), child's name (if mentioned), action description

**Signal feed**
- Clean web UI showing only `action_required` and `reminder` items
- Sorted by urgency / due date
- Mark as done, snooze, or dismiss
- Full email accessible on click

**Notifications**
- Daily digest (morning, configurable time)
- Immediate push for `action_required` items
- SMS fallback option (Twilio)

**Multi-child support**
- Tag signals by child
- Filter feed by child

### Phase 2

- Mobile app (React Native or PWA)
- Auto-create calendar events from extracted dates
- Auto-create reminders in Apple Reminders / Google Tasks
- Attachment handling (permission slips, PDFs — surface and store)
- Weekly summary report per child
- Shared access (two parents, one account)

### Phase 3

- School / district portal (structured data push, no email scraping needed)
- Teacher-side: send structured messages with metadata (type, due date, child)
- WhatsApp / Telegram delivery option
- Multilingual support
- District-level licensing / B2B

---

## Data Model

### Users
- id, email, name, timezone, created_at
- notification_preferences (digest time, push enabled, SMS number)

### Children
- id, user_id, name, school_name, grade, school_email_domains[]

### Email Connections
- id, user_id, provider (gmail/outlook/imap), oauth_token, last_synced_at, status

### Signals
- id, user_id, child_id, raw_email_id
- classification (action_required / reminder / fyi / ignore)
- title (AI-extracted summary, 1 line)
- body_excerpt
- due_date (nullable, extracted)
- action_description (nullable, extracted)
- status (new / snoozed / done / dismissed)
- created_at, updated_at

### Raw Emails
- id, connection_id, external_message_id, from, subject, body_text, received_at
- stored separately from signals — signals are the processed layer

---

## AI Classification Design

**Model:** Claude claude-haiku-4-5 (fast, cheap, accurate enough for classification)

**Input per email:**
- From address
- Subject line
- Body text (truncated to 2000 chars)
- Child names on account (for attribution)

**Output (structured JSON):**
```json
{
  "classification": "action_required",
  "title": "Permission slip due for field trip to Science Museum",
  "due_date": "2026-03-28",
  "child": "Grayson",
  "action": "Sign and return permission slip",
  "confidence": 0.94
}
```

**Cost estimate:** ~$0.0002 per email classified. At 20 emails/week/user = $0.004/month/user. Negligible.

---

## Tech Stack

- **Frontend:** React + Vite (web), React Native or PWA (mobile Phase 2)
- **Backend:** FastAPI (Python) — better AI library support than Node
- **Database:** PostgreSQL (Railway)
- **Email processing:** Gmail API, Microsoft Graph API, imaplib (Python)
- **AI:** Anthropic API (claude-haiku-4-5 for classification, claude-sonnet-4-5 for complex extraction)
- **Notifications:** Firebase Cloud Messaging (push), Twilio (SMS)
- **Hosting:** Vercel (frontend) + Railway (backend + DB)
- **Auth:** Supabase Auth or Auth0 (don't build this ourselves)

---

## Pricing

**Free tier:**
- 1 child
- Gmail only
- Daily digest only (no push)
- 30-day history

**Pro — $4.99/month or $39/year:**
- Up to 4 children
- All email providers
- Push notifications + SMS
- Unlimited history
- Shared access (2 parents)
- Calendar integration

**Family — $9.99/month (future):**
- Unlimited children
- Priority support
- Early access to new features

---

## Compliance + Privacy (Critical)

- **FERPA (US):** School records are protected. We process parent's *own* email inbox — not school systems directly. Parent consents to their own email access. This is legally similar to a personal email assistant.
- **COPPA:** We don't collect data *from* children. We collect data *about* school communications *for* parents. Distinguish clearly.
- **GDPR (EU/Canada):** Data processing agreement, right to deletion, data residency options needed for EU launch.
- **Data retention:** Raw email bodies deleted after 90 days. Signals kept as long as account is active.
- **Never store:** OAuth tokens unencrypted. Passwords. Full email bodies beyond retention window.
- **Security:** Encrypt tokens at rest, HTTPS only, SOC2 eventually (needed for school district deals).

---

## Explicitly Out of Scope (v1)

- School-side portal or teacher tools
- Reading emails from apps (ClassDojo, Bloomz, Seesaw) — email only in v1
- Replying to emails through the app
- Translating emails
- Android push notifications (iOS PWA first)

---

## Open Questions

1. **Product name:** SchoolSignal is a working name. Worth pressure-testing.
2. **iOS App Store:** PWA first or native? Apple's PWA push notification support is now decent on iOS 16.4+.
3. **Google OAuth verification:** Google requires app review for Gmail API access at scale. Needs to be started early — can take weeks.
4. **Monetization timing:** Free during beta, charge at launch or after N users?
5. **Jason + Ben involvement:** As users/advisors, or want in on the build eventually?

---

## Go-To-Market (Early)

1. Build for Grayson. Make it work perfectly for one real case.
2. Onboard Jason, Ben, and 5-10 parents manually (beta). White-glove setup.
3. Collect signal on what classifications are wrong, what's missing.
4. Launch Product Hunt when v1 is solid.
5. Parent Facebook groups, school community boards — organic to start.

---

## Success Metrics (Phase 1)

- 10 beta users in month 1
- Classification accuracy >90% (measured by user corrections)
- Zero missed `action_required` emails (the one failure mode that kills trust)
- Average daily active rate >60% (parents checking their feed)

---

## Build Sequence

1. Email connection + sync (Gmail first)
2. Classification pipeline (Haiku, structured output)
3. Signal feed UI (web)
4. Daily digest notification
5. Multi-child tagging
6. Outlook + iCloud support
7. Push notifications
8. Paywall + Stripe

*Estimated Phase 1 build time: 3-4 weeks with focused effort.*
