# AFM Ireland — Galway Assembly E-Calendar
## Project Summary, Implementation Framework & System Architecture

---

## 1. PROJECT FRAMING

### 1.1 Context & Need
The Apostolic Faith Mission (AFM) Ireland — Galway Assembly is a small congregation of **19 members** (5 families) meeting every Sunday at the Galway Westside Community Hall (H91 R8S3). The church needed a centralised, accessible, year-long digital calendar to:

- Display all church events, departmental activities, and national conferences for the current year
- Automate the allocation of **Sunday preaching and facilitation duties** fairly across members
- Provide a **confirmation workflow** via WhatsApp for assigned members
- Allow the Secretary and Pastor to manage, delegate, and publish the calendar
- Be accessible to all church members on any device via a single URL

### 1.2 Design Constraints
| Constraint | Decision |
|---|---|
| **Single-file deployment** | Entire application in one `index.html` (HTML + CSS + JS) — no build tools, no server |
| **No backend** | All state in browser `localStorage`; publishing via GitHub API from the client |
| **Mobile-first** | 85%+ of members access via smartphone — responsive design, touch gestures, large tap targets |
| **Offline-resilient** | Published page works with baked-in data; no runtime API calls needed for viewing |
| **Low technical skill** | WhatsApp-based interaction for confirming duties; no app install required |

### 1.3 Stakeholder Roles
| Role | Person(s) | Access Level |
|---|---|---|
| **Church Member** | All 19 members | View calendar, events, assignments (public) |
| **Preacher/Facilitator** | Assigned members each Sunday | Respond via WhatsApp CONFIRM/RESCHEDULE/DECLINE |
| **Secretary** | Mr T Ncube | Full admin: manage assignments, publish, delegate, backup |
| **Pastor** | Pastor C Mudada | Admin: view schedule, receive delegation, manage in Secretary's absence |

---

## 2. IMPLEMENTATION FRAMEWORK

Each requirement is listed with its resolution and the technical approach taken.

---

### 2.1 CALENDAR CORE

| Requirement | How Addressed |
|---|---|
| **Year-long church calendar** | `generateEvents(year)` dynamically populates events for any year — departmental, national, fasting, prayer, and special occasions. Year is determined at runtime from the system clock. |
| **Multiple views** | 4 view modes: Monthly (default), Quarterly, Half-Year, Full Year — rendered by `setView()` and `renderMultiMonth()` |
| **Navigation** | Arrow buttons ◀ ▶, keyboard arrows, and touch swipe (80px threshold) |
| **Monthly devotional themes** | Each month has a theme and Bible verse displayed via `renderMonthVerse()` — e.g., January = "New Beginnings" (Isaiah 43:19) |
| **Event colour coding** | 7 categories with coloured dots: General (pink), Ladies (red), Men (blue), Youth (amber), Sunday School (green), National (dark green), Prayer (grey) |
| **Department filters** | `setFilter()` toggles `activeFilter`; `filterEvents()` checks `ev.depts.includes(activeFilter)` — works across all views |
| **Multi-department events** | Events have a `depts[]` array — e.g., National Ladies Conference appears under both "Ladies" and "National" filters |
| **Day detail modal** | Tap any date → `openDayModal()` shows events, activity descriptions, and Sunday assignment details with confirmation controls |
| **Calendar watermark** | Semi-transparent AFM logo SVG behind the calendar grid for visual identity |

**Key technical decisions:**
- Events generated programmatically (not hardcoded JSON) to handle recurring patterns (every Monday prayer, quarterly special Sundays)
- `depts[]` array replaced the original single `dept` string to solve cross-department filtering for national events
- Year is fully dynamic via `currentYear` variable — all display text, WhatsApp messages, modals, and allocation use it

---

### 2.2 SUNDAY DUTY ALLOCATION ENGINE

| Requirement | How Addressed |
|---|---|
| **Fair distribution** | Deficit-based balanced algorithm — `pickCandidate()` always selects the most under-served member relative to their target |
| **Age-appropriate weighting** | `getMemberTier()` classifies members as adult/young-adult/youth/child; weights differ per tier (e.g., Pastor preaches at weight 28, adults at 2.5, children at 0) |
| **Pastor scheduling** | Soft cap of 2 Sundays/month, hard cap of 3; no consecutive Sundays (`pickCandidate` enforces `lastId` exclusion) |
| **Special Sundays** | `getSpecialSundayDept()` identifies department-led Sundays; `getDeptLeader()` and `getDeptMembers()` ensure only that department's members serve — for that ONE Sunday only, not the whole month |
| **No preacher = facilitator overlap** | `pickCandidate()` excludes the selected preacher ID when picking the facilitator |
| **College student (Panayote)** | `isPanayoteAvailable(month)` limits availability to June–August for college breaks |

**Allocation weight table:**

| Tier | Preacher Weight | Facilitator Weight |
|---|---|---|
| Pastor | 28 | 2 (light facilitation) |
| Adult | 2.5 | 3.5 |
| Young Adult (18–20) | 1.5 | 2.5 |
| Youth (14–17) | 0 (cannot preach) | 2 |
| Child (<13) | 0 | 0 |

**Algorithm flow:**
1. Compute total pool weights for preachers and facilitators separately
2. Derive per-person target count proportional to their weight
3. For each of 52 Sundays, check if it's a Special Sunday
   - **Special:** Use department members only, rotate facilitator via counter
   - **Normal:** Use `pickCandidate()` — sort pool by deficit ratio (`count/target`), skip consecutive repeat, pick the most under-served
4. Update running counts; store in `assignments` object

---

### 2.3 CONFIRMATION WORKFLOW

| Requirement | How Addressed |
|---|---|
| **WhatsApp-based confirmation** | `sendConfirmationRequest()` opens `wa.me/{phone}?text=...` with compact pre-filled message containing 3 clickable reply links |
| **Three response options** | CONFIRM ✅, RESCHEDULE 🔄, and DECLINE ❌ — each is a `wa.me` link that sends a pre-filled reply back to the Secretary/Pastor |
| **Reschedule limit** | Max 2 reschedules per assignment, tracked via `rescheduleCount` |
| **Decline → alternatives** | `showAlternatives()` lists eligible replacements, sorted by deficit (least-assigned first), with count badges |
| **Assignment status tracking** | Each duty has status: `pending` → `confirmed` / `declined` / `locked` |
| **Manual editing** | `editAssignmentRole()` allows Secretary to directly swap any preacher or facilitator |

**WhatsApp message format:**
```
AFM Galway
📖 Preacher: 2 Feb
You: Pastor C Mudada

✅ CONFIRM → [wa.me link]
🔄 RESCHEDULE → [wa.me link]
❌ DECLINE → [wa.me link]
```

---

### 2.4 DELEGATION SYSTEM

| Requirement | How Addressed |
|---|---|
| **Secretary absence** | `toggleDelegation()` flips `delegated_to_pastor` in localStorage |
| **WhatsApp rerouting** | `getActivePhone()` returns Pastor's number when delegated, Secretary's otherwise |
| **Visual indicator** | `updateDelegationUI()` changes the delegation button text and style |
| **Persistent** | Delegation state survives page reload (stored in localStorage) |

---

### 2.5 PUBLISHING & HOSTING

| Requirement | How Addressed |
|---|---|
| **Live website** | Hosted on GitHub Pages at `https://tuls78.github.io/eCalGalway/` |
| **One-click publish** | `publishToGitHub()` pushes directly to GitHub via Personal Access Token |
| **Data baking** | `buildPublishableHTML()` fetches clean source, replaces `// __BAKE_POINT__` marker with serialised assignment data |
| **Re-publish safe** | Marker-based replacement works for both first publish and subsequent updates |
| **Token persistence** | GitHub token stored in localStorage, preserved across resets |
| **Clean source fetch** | `fetch(window.location.href)` retrieves original HTML to avoid DOM serialisation snapshot issues (fallback: `outerHTML` with state cleanup) |

**Publish flow:**
```
Secretary clicks 🚀 Publish
  → GET /repos/{owner}/{repo}/contents/index.html (fetch SHA)
  → buildPublishableHTML() (fetch clean source, bake assignments)
  → utf8ToBase64(html) (encode for GitHub API)
  → PUT /repos/{owner}/{repo}/contents/index.html (push update)
  → GitHub Pages rebuilds (~60 seconds)
```

---

### 2.6 DATA PERSISTENCE & BACKUP

| Requirement | How Addressed |
|---|---|
| **Local storage** | All assignments saved to `localStorage` key `afm_assignments` via `saveAssignments()` |
| **Export** | `exportData()` downloads JSON file with all assignment data |
| **Import** | `importDataPrompt()` reads JSON file and restores assignments |
| **Smart reset** | `resetAssignmentsOnly()` clears assignments but preserves settings keys: `gh_token`, `gh_owner`, `gh_repo`, `gh_file`, `secretary_phone`, `pastor_phone`, `delegated_to_pastor` |

---

### 2.7 SECURITY & ACCESS CONTROL

| Requirement | How Addressed |
|---|---|
| **Password protection** | `toggleAdmin()` prompts for password; validates against hardcoded values |
| **Two roles** | `"Shepherd26"` → Pastor, `"Galway26"` → Secretary |
| **Session-only auth** | `isAdminLoggedIn` resets on page reload — no persistent login |
| **Role-based Help** | `showHelp()` checks `isAdminLoggedIn` and `adminRole` to show only relevant sections |

**Help access matrix:**

| User State | Visible Help Sections |
|---|---|
| Not logged in (Member) | 👥 Member only |
| Logged in as Pastor | 👥 Member + 📱 WhatsApp + ⛪ Pastor |
| Logged in as Secretary | 👥 Member + 📱 WhatsApp + 📋 Secretary + ⛪ Pastor |

---

### 2.8 ALLOCATION MATRIX & ANALYTICS

| Requirement | How Addressed |
|---|---|
| **Duty balance visibility** | `showAllocationMatrix()` renders a table showing actual vs target counts per member |
| **Visual indicators** | Colour-coded bars: Green = balanced, Orange = under-served, Red = over-served |
| **Split view** | Separate tables for Preachers and Facilitators |
| **Pastor breakdown** | Monthly preaching count for the Pastor displayed separately |
| **Target computation** | `computeTargets()` derives expected duties from weight proportions across all normal Sundays |

---

### 2.9 WHATSAPP SHARING

| Requirement | How Addressed |
|---|---|
| **Monthly rota sharing** | `shareMonthCalendar()` builds a text summary of the month's events and Sunday assignments, opens WhatsApp share |
| **Day-level sharing** | `buildDayShareText()` generates a focused view for a specific date |
| **Share button** | Fixed 📱 FAB button accessible from all views |

---

### 2.10 SPECIAL SUNDAYS — DEPARTMENT-LED SERVICES

| Requirement | How Addressed |
|---|---|
| **Quarterly rotation** | Each department leads one Sunday per quarter via `getSpecialSundayDept()` |
| **Department-only participation** | Only that department's members are eligible for preaching and facilitation on their Special Sunday |
| **Leader priority** | `getDeptLeader()` gives automatic preaching assignment to department heads |
| **No spillover** | Special Sunday assignment is for that ONE day only — not the entire month |

**Department leaders:**
| Department | Leader | Preaching Priority |
|---|---|---|
| Ladies | Mrs Pastor L Mudada | First preacher |
| Youth | Mr V Gill | First preacher |
| Sunday School | Mrs S Gill | Only preacher; children may facilitate |
| Men | Mr Dube | Rotates with Mr T Ncube and Mr V Gill |

---

### 2.11 NATIONAL VS LOCAL EVENTS

| Requirement | How Addressed |
|---|---|
| **No service cancellation** | National events (conferences, camps) don't cancel local Sunday services — most members cannot travel |
| **Dual tagging** | National events use `depts: ["ladies", "national"]` etc., visible under both filters |
| **No allocation conflict** | Normal rota continues on Sundays that fall during national events |

---

### 2.12 ACCESS CONTROL — CONFIRMATION WORKFLOW

| Requirement | How Addressed |
|---|---|
| **Admin-only confirmation** | `buildConfirmationUI()` checks `isAdminLoggedIn` — only Secretary/Pastor see action buttons |
| **Read-only for members** | Church members see assignment names and status badges (✅ Confirmed, ⏳ Pending, ❌ Declined) but no Confirm/Reschedule/Decline buttons |
| **Guard on all actions** | Every action function (`confirmAssignment`, `rescheduleAssignment`, `declineAssignment`, `showAlternatives`, `assignAlternative`, `sendConfirmationRequest`, `editAssignmentRole`) checks `isAdminLoggedIn` and returns early with a toast if not authorised |
| **Informative guidance** | Non-admin modal shows: "🔒 Only the Secretary or Pastor can manage confirmations. Log in via ⚙ to access controls." |

---

### 2.13 ROBUSTNESS & SAFETY

| Requirement | How Addressed |
|---|---|
| **Safe reset** | `resetAssignmentsOnly()` removes only `afm_assignments` key — does NOT wipe entire localStorage (protects other apps on same domain) |
| **Allocation crash prevention** | `pickCandidate()` has a final fallback: if the candidate pool is empty after all filters, it falls back to the full pool or defaults to Pastor (ID 1) to prevent `undefined` errors |
| **Toast notification queue** | `showToast()` uses a queue-based system — rapid actions no longer overwrite each other; messages display sequentially (3s each) |
| **Security transparency** | Plaintext password system explicitly documented with code comment: *"Client-side simple authentication. Not for high-security data."* |

---

### 2.14 YEAR ROLLOVER & FUTURE-PROOFING

| Requirement | How Addressed |
|---|---|
| **Dynamic year** | `currentYear` is set from `new Date().getFullYear()` at runtime — all display text, WhatsApp messages, modals, allocation, and sharing use it dynamically |
| **Year-keyed assignments** | `localStorage` key `afm_assignments` holds data per-year; resetting clears only assignments, preserving settings |
| **November reminder** | `checkYearEndReminder()` fires from 15 November after Secretary/Pastor login — modal with 6-step preparation guide for the next year |
| **Reminder access control** | Only appears for `isAdminLoggedIn` users (Secretary/Pastor) — never shown to church members |
| **Dismiss / Snooze** | "Got it — Dismiss" stores a per-year key in localStorage; "Remind Me Later" closes but returns on next login |
| **Event extensibility** | `generateEvents(year)` accepts any year; add a new `if (year === 2027)` block with that year's dates |

---

## 3. SYSTEM ARCHITECTURE

### 3.1 High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    SINGLE HTML FILE                          │
│                     (index.html)                             │
│                                                              │
│  ┌──────────────┐ ┌──────────────┐ ┌───────────────────┐    │
│  │   CSS Layer   │ │  HTML Layer  │ │  JavaScript Layer │    │
│  │  (~1,055 ln)  │ │  (~310 ln)   │ │   (~2,000 ln)     │    │
│  └──────────────┘ └──────────────┘ └───────────────────┘    │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                JAVASCRIPT ENGINE                      │   │
│  │                                                       │   │
│  │  ┌─────────┐  ┌──────────┐  ┌──────────────────┐    │   │
│  │  │  DATA    │  │  RENDER  │  │  ADMIN WORKFLOW  │    │   │
│  │  │  LAYER   │  │  ENGINE  │  │  ENGINE          │    │   │
│  │  │         │  │          │  │                  │    │   │
│  │  │ MEMBERS │  │ Calendar │  │ Confirmations   │    │   │
│  │  │ EVENTS  │  │ Modals   │  │ Alternatives    │    │   │
│  │  │ ASSIGNS │  │ Filters  │  │ Delegation      │    │   │
│  │  │ THEMES  │  │ Views    │  │ Allocation Mtx  │    │   │
│  │  └────┬────┘  └────┬─────┘  └────────┬─────────┘    │   │
│  │       │             │                  │              │   │
│  │  ┌────▼─────────────▼──────────────────▼──────────┐  │   │
│  │  │              STATE MANAGEMENT                   │  │   │
│  │  │  currentMonth | activeFilter | assignments     │  │   │
│  │  │  currentView  | adminRole    | isAdminLoggedIn │  │   │
│  │  └────────────────────┬───────────────────────────┘  │   │
│  │                       │                               │   │
│  │  ┌────────────────────▼───────────────────────────┐  │   │
│  │  │           EXTERNAL INTERFACES                   │  │   │
│  │  │                                                 │  │   │
│  │  │  localStorage    GitHub API     WhatsApp API   │  │   │
│  │  │  (persistence)   (publishing)   (messaging)    │  │   │
│  │  └─────────────────────────────────────────────────┘  │   │
│  └───────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        PRESENTATION LAYER                       │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌─────────┐  ┌───────────────┐   │
│  │  Header   │  │Calendar  │  │  Modal   │  │  Admin Panel  │   │
│  │  + Logo   │  │  Grid    │  │  System  │  │  (Secretary)  │   │
│  │  + Verse  │  │  + Dots  │  │  + Day   │  │  + Buttons    │   │
│  │           │  │  + Nav   │  │  + Help  │  │  + Settings   │   │
│  │           │  │  + Tabs  │  │  + Alloc │  │  + Delegate   │   │
│  │           │  │  + Fltrs │  │  + Pend  │  │  + Matrix     │   │
│  └──────────┘  └──────────┘  └─────────┘  └───────────────┘   │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌─────────┐  ┌───────────────┐   │
│  │  Legend   │  │ Event    │  │  Toast   │  │  FAB Buttons  │   │
│  │  (7 cats) │  │ List     │  │  Notifs  │  │  ⚙ ? 📱      │   │
│  └──────────┘  └──────────┘  └─────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         LOGIC LAYER                             │
│                                                                 │
│  ┌─────────────────┐  ┌────────────────┐  ┌─────────────────┐  │
│  │  ALLOCATION      │  │  CONFIRMATION  │  │  PUBLISHING     │  │
│  │  ENGINE          │  │  WORKFLOW      │  │  ENGINE         │  │
│  │                  │  │                │  │                 │  │
│  │ getMemberTier()  │  │ confirmAssign()│  │ buildPublish()  │  │
│  │ getPreachWt()    │  │ reschedule()   │  │ utf8ToBase64()  │  │
│  │ getFacilWt()     │  │ decline()      │  │ publishGitHub() │  │
│  │ pickCandidate()  │  │ showAltern()   │  │ getGHConfig()   │  │
│  │ generateInit()   │  │ assignAlt()    │  │ setupGitHub()   │  │
│  │ computeTargets() │  │ sendConfReq()  │  │ saveGHSettings()│  │
│  │ computeCounts()  │  │ editAssign()   │  │                 │  │
│  └─────────────────┘  └────────────────┘  └─────────────────┘  │
│                                                                 │
│  ┌─────────────────┐  ┌────────────────┐  ┌─────────────────┐  │
│  │  SPECIAL SUNDAY  │  │  DELEGATION    │  │  HELP SYSTEM    │  │
│  │  LOGIC           │  │  SYSTEM        │  │                 │  │
│  │                  │  │                │  │ showHelp()      │  │
│  │ getSpcSunDept()  │  │ toggleDeleg()  │  │ showHelpSect()  │  │
│  │ getDeptLeader()  │  │ updateDelUI()  │  │ (role-based     │  │
│  │ getDeptMembers() │  │ getActivePhone │  │  access matrix) │  │
│  │ isPanAvail()     │  │ isDelegated()  │  │                 │  │
│  └─────────────────┘  └────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                             │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │  MEMBERS[19] │  │  EVENTS      │  │  ASSIGNMENTS       │    │
│  │              │  │  (generated) │  │  (localStorage)    │    │
│  │  id, name    │  │              │  │                    │    │
│  │  role, dept  │  │  month, day  │  │  "M-D" → {        │    │
│  │  canPreach   │  │  title       │  │    preacher: {     │    │
│  │  canFacilite │  │  dept, depts │  │      memberId,     │    │
│  │  available   │  │  key         │  │      status,       │    │
│  │  age         │  │  endDay      │  │      reschedCnt    │    │
│  │              │  │              │  │    },              │    │
│  │              │  │              │  │    facilitator: {} │    │
│  ├──────────────┤  ├──────────────┤  │  }                 │    │
│  │ACTIVITY_INFO │  │MONTHLY_THEMES│  └────────────────────┘    │
│  │  (12 acts)   │  │  (12 months) │                            │
│  └──────────────┘  └──────────────┘                            │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Data Flow Architecture

```
                        ┌──────────────────┐
                        │    PAGE LOAD     │
                        └────────┬─────────┘
                                 │
                    ┌────────────▼────────────┐
                    │ generateInitialAssign() │
                    │                         │
                    │  localStorage exists?   │
                    │    YES → parse & return │
                    │    NO  → run algorithm  │
                    │         → save to LS    │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │       init()            │
                    │  Set currentMonth       │
                    │  renderCalendar()       │
                    │  updateDelegationUI()   │
                    └────────────┬────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
    ┌─────▼─────┐        ┌──────▼──────┐       ┌───────▼──────┐
    │  USER     │        │   ADMIN     │       │  WHATSAPP    │
    │  BROWSING │        │   ACTIONS   │       │  RESPONSE    │
    │           │        │             │       │              │
    │ Navigate  │        │ Confirm     │       │ Member taps  │
    │ Filter    │        │ Reschedule  │       │ CONFIRM link │
    │ Tap date  │        │ Decline     │       │ → reply sent │
    │ View modes│        │ Publish     │       │ → Secretary  │
    │ Share     │        │ Delegate    │       │   updates    │
    └───────────┘        │ Reset       │       │   status     │
                         │ Export/Imp  │       └──────────────┘
                         └──────┬──────┘
                                │
                    ┌───────────▼───────────┐
                    │    saveAssignments()  │
                    │    localStorage.set   │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    PUBLISH FLOW       │
                    │                       │
                    │ fetch clean source    │
                    │ bake assignments      │
                    │ base64 encode         │
                    │ GitHub PUT API        │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    GITHUB PAGES       │
                    │                       │
                    │ tuls78.github.io/     │
                    │   eCalGalway/         │
                    │                       │
                    │ Static HTML with      │
                    │ baked-in data         │
                    │ Fully interactive     │
                    └───────────────────────┘
```

### 3.4 Security Architecture

```
┌───────────────────────────────────────────┐
│            ACCESS CONTROL MODEL           │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │         PUBLIC ACCESS               │  │
│  │  • View calendar (all views)        │  │
│  │  • Browse events                    │  │
│  │  • Use department filters           │  │
│  │  • Share via WhatsApp               │  │
│  │  • View Help (Member section only)  │  │
│  └─────────────────────────────────────┘  │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │    PASTOR ACCESS (Shepherd26)       │  │
│  │  • All public features             │  │
│  │  • Admin Panel (when delegated)     │  │
│  │  • Receive WhatsApp replies         │  │
│  │  • Help: Member + WhatsApp + Pastor │  │
│  └─────────────────────────────────────┘  │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │   SECRETARY ACCESS (Galway26)      │  │
│  │  • All public features             │  │
│  │  • Full Admin Panel                │  │
│  │  • Publish to GitHub               │  │
│  │  • Manage assignments              │  │
│  │  • Delegate to Pastor              │  │
│  │  • Export / Import / Reset data    │  │
│  │  • Allocation Matrix               │  │
│  │  • Help: ALL sections              │  │
│  └─────────────────────────────────────┘  │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │        SECURITY NOTES              │  │
│  │  • Session-only auth (no cookies)  │  │
│  │  • Plaintext passwords in source   │  │
│  │  • GitHub token in localStorage    │  │
│  │  • Suitable for trusted small      │  │
│  │    congregation use case           │  │
│  └─────────────────────────────────────┘  │
└───────────────────────────────────────────┘
```

### 3.5 Allocation Engine Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   ALLOCATION ENGINE                         │
│                                                             │
│  INPUT                                                      │
│  ├── MEMBERS[19] with canPreach, canFacilitate, dept, age  │
│  ├── ALL_SUNDAYS_2026 (52 Sundays)                         │
│  └── Special Sunday schedule (quarterly per dept)           │
│                                                             │
│  STEP 1: CLASSIFY                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ getMemberTier(m) → adult | young-adult | youth | child │
│  │ getPreacherWeight(m) → 0 / 1 / 1.5 / 2.5 / 28       │
│  │ getFacilitatorWeight(m) → 0 / 2 / 2.5 / 3.5          │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  STEP 2: BUILD POOLS                                       │
│  ┌──────────────────┐  ┌──────────────────┐                │
│  │  Preacher Pool   │  │ Facilitator Pool │                │
│  │  (weight > 0)    │  │  (weight > 0)    │                │
│  │  + target count  │  │  + target count  │                │
│  │  + running count │  │  + running count │                │
│  │  + monthly count │  │  + monthly count │                │
│  └──────────────────┘  └──────────────────┘                │
│                                                             │
│  STEP 3: ITERATE 52 SUNDAYS                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  For each Sunday:                                    │   │
│  │    Is Special Sunday?                                │   │
│  │    ├── YES → getDeptLeader() + getDeptMembers()     │   │
│  │    │         Use department-only pool                │   │
│  │    └── NO  → pickCandidate(preacherPool)            │   │
│  │              pickCandidate(facilitatorPool)          │   │
│  │              Rules:                                  │   │
│  │              • No consecutive same person            │   │
│  │              • Preacher ≠ Facilitator                │   │
│  │              • Pastor: soft cap 2/mo, hard cap 3/mo │   │
│  │              • Select by lowest deficit ratio        │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  OUTPUT                                                     │
│  └── assignments{} → 52 entries, each with:                │
│       preacher: { memberId, status, rescheduleCount }      │
│       facilitator: { memberId, status, rescheduleCount }   │
└─────────────────────────────────────────────────────────────┘
```

### 3.6 Publishing Pipeline Architecture

```
LOCAL BROWSER                           GITHUB                    END USER
─────────────                           ──────                    ────────

┌──────────────┐                   ┌──────────────┐
│ Secretary    │                   │  GitHub API  │
│ clicks 🚀   │                   │              │
└──────┬───────┘                   │ repos/tuls78 │
       │                           │ /eCalGalway/ │
       ▼                           │ contents/    │
┌──────────────┐    GET SHA        │ index.html   │
│ publishTo    ├──────────────────►│              │
│ GitHub()     │◄──────────────────┤ returns SHA  │
│              │                   │              │
│              │                   │              │
│ buildPublish │                   │              │
│ ableHTML()   │                   │              │
│  │           │                   │              │
│  ├─fetch src │                   │              │         ┌──────────────┐
│  ├─bake data │    PUT (base64)   │              │         │  GitHub      │
│  ├─base64    ├──────────────────►│  commit      │────────►│  Pages       │
│  └─encode    │                   │  updated     │  build  │              │
│              │◄──── 200 OK ──────┤              │         │ tuls78.      │
│              │                   └──────────────┘         │ github.io/   │
│ showToast ✅ │                                            │ eCalGalway/  │
└──────────────┘                                            │              │
                                                            │ Static HTML  │
                                                      ┌────►│ + baked data │
                                                      │     │ Interactive  │
                                                      │     └──────────────┘
                                                      │
                                              ┌───────┴───────┐
                                              │ Church members│
                                              │ open URL on   │
                                              │ any device    │
                                              └───────────────┘
```

---

## 4. FILE INVENTORY

| File | Size | Purpose |
|---|---|---|
| `index.html` | ~3,490 lines | Complete application (HTML + CSS + JS) |
| `README.md` | ~200 lines | GitHub repository documentation |
| `PROJECT_SUMMARY.md` | This file | Framing, implementation, and architecture document |

---

## 5. TECHNOLOGY STACK

| Layer | Technology | Purpose |
|---|---|---|
| **Markup** | HTML5 | Structure, semantic elements |
| **Styling** | CSS3 (inline `<style>`) | Responsive layout, animations, print styles |
| **Logic** | Vanilla JavaScript (ES6+) | All application logic, no frameworks |
| **Graphics** | Inline SVG | AFM logo, watermark, crown/cross symbolism |
| **Storage** | Web localStorage | Assignment data, settings, token persistence |
| **Hosting** | GitHub Pages | Static site hosting, CDN delivery |
| **API** | GitHub REST API v3 | Direct file push/update for publishing |
| **Messaging** | WhatsApp URL scheme (`wa.me`) | Duty confirmations, calendar sharing |
| **Fonts** | Google Fonts (Playfair Display, Inter) | Typography |

---

## 6. EVOLUTION LOG

| Phase | What Was Built | Key Decision |
|---|---|---|
| 1. Core Calendar | Month view, events, navigation | Single HTML file for zero-dependency deployment |
| 2. Logo & Branding | Inline SVG with AFM crown/cross/rays | SVG-in-HTML avoids external image dependency |
| 3. Security | Password-protected admin panel | Session-only auth (no server backend) |
| 4. GitHub Publishing | One-click push to live site | Client-side GitHub API via Personal Access Token |
| 5. WhatsApp Integration | Confirmation workflow with clickable links | wa.me URL scheme — no app registration needed |
| 6. Delegation | Secretary → Pastor handover | localStorage flag + phone number rerouting |
| 7. Smart Reset | Preserve tokens on reset | Selective localStorage clearing |
| 8. WhatsApp UX | Compact message format | Shortened labels: CONFIRM/RESCHEDULE/DECLINE |
| 9. Special Sundays | Department-led services | One-Sunday-only scope, dept member pools |
| 10. Balanced Allocation | Deficit-based engine | Replaced simple rotation with weighted fairness |
| 11. Allocation Matrix | Visual duty balance dashboard | Actual vs target with colour-coded bars |
| 12. Multi-Dept Filtering | depts[] array on events | Solved national events appearing under both filters |
| 13. All-View Filters | Filters work in multi-month views | renderMultiMonth() now respects activeFilter |
| 14. Documentation | README.md + in-app Help | Role-based access matrix for Help content |
| 15. Publish Fix | Clean source fetch + bake marker | Fixed frozen snapshot issue with __BAKE_POINT__ |
| 16. Admin Panel UX | Close button + click-outside | Panel no longer stays open unexpectedly |
| 17. Access Control | Role-gated confirmation workflow | Only Secretary/Pastor can Confirm/Reschedule/Decline; members see read-only status |
| 18. Safe Reset | Targeted `localStorage.removeItem` | No longer wipes entire domain storage |
| 19. Crash Prevention | `pickCandidate()` fallback | Empty pool defaults to Pastor instead of crashing |
| 20. Toast Queue | Sequential notification display | Rapid actions no longer overwrite each other |
| 21. Dynamic Year | `currentYear` from system clock | All 40+ hardcoded "2026" references replaced with dynamic variable |
| 22. Year Rollover | November reminder for admin | `checkYearEndReminder()` fires on Secretary/Pastor login from 15 Nov |

---

*Document updated: 15 February 2026*
*Platform: AFM Ireland Galway Assembly E-Calendar v1.1*
*Repository: https://github.com/tuls78/eCalGalway*
*Live URL: https://tuls78.github.io/eCalGalway/*
