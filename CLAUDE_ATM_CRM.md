# CLAUDE.md — ATM Brokerage CRM (Deal Room Concierge)

## What this project is
The ATM Brokerage CRM is an internal deal management and AI email agent system for ATM Brokerage. It handles buyer email triage, AI-drafted responses in John's voice, deal tracking, and a token-gated buyer portal (Deal Hub).

- Live at: `atm-brokerage-crm.vercel.app`
- Deploys to: Vercel (Next.js App Router)
- Primary users: Jeff (super user), John (super user), future broker advisors (broker role)

---

## Supabase Project

| Project | ID | Purpose |
|---|---|---|
| ATM CRM | `wgrmxhxozoyvcmvbfuxv` | CRM data, deals, buyer tokens, email state |

Always use `SUPABASE_URL` and `SUPABASE_SERVICE_KEY` env vars. Never hardcode credentials.

---

## Key Tables

- `deal_tokens` — token-gated buyer portal access; written to by NDA webhook
- `deals` — deal pipeline and tracking
- Supabase Auth handles user roles: `super` (Jeff, John) and `broker` (future advisors)

---

## Gmail Integration

### OAuth setup
- Scope: `gmail.modify` (required — do not downgrade)
- Tokens stored as pickle files on Jeff's Mac — must run locally, not on Vercel
- `token_john.pickle` — **not yet created**; John's OAuth flow still needs to be completed
- Run order for daily workflow:
  1. `gmail_sync.py` — pulls new emails from info@ and john@ inboxes
  2. Review + draft in Deal Room panel (`app/crm.js`)
  3. `gmail_draft_push.py` — pushes approved drafts back to Gmail

### Inboxes
- `info@` — general ATM Brokerage inquiries
- `john@` — John's deal inbox

### Critical notes
- Gmail OAuth tokens are local-only (Jeff's Mac). Do not attempt to run Gmail sync from Vercel or any remote environment.
- If token auth fails, re-run the OAuth flow locally to refresh.

---

## AI Email Agent (Deal Room Concierge)

### How it works
1. `gmail_sync.py` pulls incoming buyer emails
2. Buyer email classifier (Claude API) categorizes intent
3. Supabase context feed enriches with deal history
4. Draft generator produces reply in **John's voice**
5. Jeff reviews drafts in Deal Room panel
6. `gmail_draft_push.py` pushes approved drafts to Gmail as drafts (not auto-sent)

### John's voice
Drafts must sound like John — conversational, direct, deal-focused. When generating drafts, reference past approved drafts to calibrate tone. Do not generate formal/corporate-sounding emails.

### Known improvements (John's feedback — not yet built)
1. Multi-deal threading — tie email threads to specific deals
2. Signature fixes — John's signature not rendering correctly
3. Buyer qualification flow — add qualification questions to early-stage replies
4. Additional items from John's list (3 more TBD)

---

## Deal Hub (Buyer Portal)
- Token-gated — buyers get access via NDA completion
- NDA webhook: `app/api/nda-webhook/route.js` — writes to `deal_tokens` table
- WPForms form ID `6387` on atmbrokerage.com triggers the webhook via `wpforms_process_complete` PHP hook

---

## ATM Brokerage Seller Intake Wizard
- Live at: `atmbrokerage.com/seller-intake-form/`
- Built as standalone HTML with CSS namespaced under `.seller-intake-page`
- 5-step intake flow
- **WordPress/Enfold strips inline `<script>` tags** — all JS must go in Code Snippets → JavaScript → Site Wide Footer
- PHP file handling (`$_FILES`, `wp_mail()`) lives in Code Snippets as a PHP snippet
- Processor dropdown options: Switch Commerce, CDS, PAI/Payment Alliance, Cardtronics/Allpoint, Other

---

## Email Lists (Mailchimp)
- ATM Brokerage: ~10,868 subscribers
- ConnectATM: ~8,313 subscribers
- Tag taxonomy: role/status/source/geo prefixes — keep consistent across both lists

---

## Common Mistakes to Avoid
- Do NOT run Gmail sync from any environment other than Jeff's Mac
- Do NOT auto-send drafted emails — always push as Gmail drafts for review
- Do NOT downgrade Gmail OAuth scope below `gmail.modify`
- Do NOT add inline `<script>` tags to WordPress/Enfold pages — use Code Snippets footer
- Do NOT touch CleaningExits or DealLedger Supabase projects from this repo
- Do NOT generate formal/corporate email drafts — must match John's conversational voice
- Do NOT forget: `token_john.pickle` still needs to be created before John's inbox syncs

---

## Daily Workflow (Quick Reference)
```
# On Jeff's Mac:
python gmail_sync.py          # Pull new emails
# → Review in Deal Room at atm-brokerage-crm.vercel.app
python gmail_draft_push.py    # Push approved drafts to Gmail
```

---

## Strategic Context (for AI reasoning)
ATM Brokerage has ~90% market share in ATM business sales (200+ transactions, $75M+). The CRM exists to handle deal volume without adding headcount. The AI email agent's job is to make John's inbox manageable — triage, draft, approve, send. The priority is speed and accuracy of response, not automation for its own sake. John reviews everything before it goes out. Never build features that remove Jeff or John from the approval loop.
