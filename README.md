# AFM Galway Assembly — 2026 Church E-Calendar

A self-contained, mobile-friendly electronic church calendar for the **Apostolic Faith Mission (AFM) Ireland — Galway Assembly**.

🌐 **Live Site:** [https://tuls78.github.io/eCalGalway/](https://tuls78.github.io/eCalGalway/)

---

## Features

- **Monthly / Quarterly / Half-Year / Full-Year** views
- **Department filters** — Ladies, Men, Youth, Sunday School, National, General
- **Preaching & Facilitation rota** with balanced allocation algorithm
- **Special Sundays** — department-led services (Ladies, Men, Youth, Sunday School)
- **WhatsApp integration** — share calendars & send availability confirmations
- **GitHub auto-publish** — push updates directly from the app
- **Password-protected admin panel** for Secretary & Pastor
- **In-app Help** — role-based guides for all users

## How It Works

This is a **single HTML file** (`index.html`) with no external dependencies. It runs entirely in the browser using HTML, CSS, and JavaScript. Data is stored in the browser's `localStorage`.

### For Church Members (Public)

1. Visit the live link above
2. Browse the calendar — tap any day to see event details
3. Use department filter buttons to view only your department's events
4. Switch between Monthly, Quarterly, Half-Year, or Full-Year views
5. Tap **📱** to share the month's calendar on WhatsApp

### For People Receiving WhatsApp Availability Requests

When the Secretary (or Pastor) sends you an availability request via WhatsApp:

1. You'll receive a short message with your assigned role and date
2. Tap one of the three links to reply:
   - **✅ CONFIRM** — You'll be there
   - **🔄 RESCHEDULE** — Propose a different date
   - **❌ DECLINE** — Unable to attend (a replacement will be found)
3. You may reschedule a maximum of 2 times per assignment

### For the Secretary

Access the **Secretary Panel** by tapping the ⚙ icon (password required).

- **View All Pending** — See all unconfirmed assignments
- **Share Month via WhatsApp** — Send the full month's rota
- **Export / Import Data** — Backup and restore assignments
- **Publish to GitHub** — Push the live calendar to the website
- **Delegation** — Hand over management to the Pastor when unavailable
- **Allocation Matrix** — View the year-end balance of duties across members
- **Reset All Data** — Regenerate assignments (settings are preserved)

### For the Pastor

Access the **Pastor Panel** by tapping the ⚙ icon (Pastor password required).

- View assignments and event details
- Receive delegated duties when the Secretary is unavailable
- Approve/confirm assignments when managing the calendar

---

## Technical Details

| Item | Detail |
|---|---|
| File | Single `index.html` (~3100 lines) |
| Dependencies | None (fully self-contained) |
| Storage | Browser localStorage |
| Hosting | GitHub Pages |
| Repo | `tuls78/eCalGalway` |

## Department Structure

| Department | Members | Leader |
|---|---|---|
| Ladies | 
| Men | 
| Youth | 
| Sunday School | 

## Allocation Rules

- **14+** can facilitate
- **18+** can preach
- No person on the same role two consecutive Sundays
- Pastor preaches ~2 Sundays/month (max 3)
- Adults = heavy allocation, Youth = moderate, SS children = very light
- Special Sundays = department members only (that one Sunday)
- By 31 December, allocation is balanced across all eligible members

---

© 2026 AFM Ireland — Galway Assembly
