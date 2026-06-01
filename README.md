# QA & Automation Developer — Skills Assessment

**Submitted by:** Maheep Ghanshani
**Email:** ghanshanimaheep04@gmail.com
**Date:** June 2, 2026
**Role:** Automation & QA Developer

---

## 📁 Repository Structure

```
qa-automation-assessment/
├── Task1_QA_Report_MaheepGhanshani.pdf   ← Download full PDF report
├── Task2_Workflow_MaheepGhanshani.json   ← Import in n8n
├── screenshots/
│   ├── canvas.png
│   ├── discord_execution.png
│   ├── bug1_garbled_username.png
│   ├── bug2_xss_vulnerability.png
│   ├── bug3_html_tag_accepted.png
│   ├── bug4_leave_empty_page.png
│   └── bug5_6_myinfo_validation.png
└── README.md
```

---

# ✅ TASK 1 — Web App QA & Debug Report

## Application Under Test

| Field | Details |
|-------|---------|
| **Application** | OrangeHRM Demo |
| **URL** | https://opensource-demo.orangehrmlive.com |
| **Type** | Human Resource Management System |
| **Test Type** | Manual QA — Functional, Security, UX Testing |
| **Tester** | Maheep Ghanshani |
| **Date** | June 2, 2026 |
| **Total Bugs** | 6 |

## Why OrangeHRM?
- Real-world HR application used by actual companies
- Complex user flows: authentication, forms, social features, file uploads
- Multiple modules to test: Recruitment, Leave, PIM, Buzz, Directory
- Publicly accessible demo — no setup required

---

## 🐛 Bug Report Table

### Bug #1 — Garbled/Corrupted Username in Buzz Posts

| Field | Details |
|-------|---------|
| **Severity** | Medium |
| **Module** | Buzz (Social Feed) |
| **Steps to Reproduce** | 1. Login to OrangeHRM 2. Go to Dashboard 3. View "Buzz Latest Posts" section |
| **Expected** | User's real name should display correctly |
| **Actual** | Username shows as `@#$%S#%$#$%#%$#%$ Joh...` — garbled symbols |
| **Suspected Cause** | Special characters in username not sanitized before display; missing character encoding (UTF-8) in the frontend rendering layer |

---

### Bug #2 — XSS Script Tag Accepted in Buzz Post ⚠️ CRITICAL

| Field | Details |
|-------|---------|
| **Severity** | **Critical** |
| **Module** | Buzz (Social Feed) |
| **Steps to Reproduce** | 1. Login 2. Go to Buzz 3. Type `<script>alert('XSS')</script>` in post box 4. Click Share |
| **Expected** | Script tag should be blocked or stripped — error message shown |
| **Actual** | Raw script tag `<script>alert('XSS')</script>` is posted and visible to all users |
| **Suspected Cause** | Zero input sanitization on Buzz post content field; no output encoding when rendering user-generated content |

---

### Bug #3 — HTML Tags Accepted in Vacancy Name Field

| Field | Details |
|-------|---------|
| **Severity** | High |
| **Module** | Recruitment → Vacancies |
| **Steps to Reproduce** | 1. Go to Recruitment 2. Click Vacancies tab 3. Click Add 4. Type `<test>` in Vacancy Name 5. Click Save |
| **Expected** | HTML tags should be rejected with validation error |
| **Actual** | `<test>` is saved successfully as vacancy name |
| **Suspected Cause** | Missing server-side HTML sanitization on Vacancy Name field; no input validation for special characters |

---

### Bug #4 — Leave Page Shows No Helpful Guidance

| Field | Details |
|-------|---------|
| **Severity** | Medium |
| **Module** | Leave → Apply |
| **Steps to Reproduce** | 1. Login as non-admin user 2. Click Leave in left menu 3. Click Apply tab |
| **Expected** | Leave application form OR a clear message explaining why leave cannot be applied with next steps |
| **Actual** | Page shows only "No Leave Types with Leave Balance" — no explanation, no action button, no guidance |
| **Suspected Cause** | Missing fallback UI for empty state; no conditional rendering logic to guide users when leave balance is zero or unassigned |

---

### Bug #5 — No Character Limit on Employee Name Field

| Field | Details |
|-------|---------|
| **Severity** | Medium |
| **Module** | My Info → Personal Details |
| **Steps to Reproduce** | 1. Go to My Info 2. Click Personal Details 3. Paste 200+ characters in First Name field 4. Click Save |
| **Expected** | Validation error: "Name cannot exceed X characters" |
| **Actual** | 200+ character string is accepted and saved without any error |
| **Suspected Cause** | No `maxlength` attribute on input field; no server-side length validation for name fields |

---

### Bug #6 — Expired Driver's License Date Accepted Without Warning

| Field | Details |
|-------|---------|
| **Severity** | Low |
| **Module** | My Info → Personal Details |
| **Steps to Reproduce** | 1. Go to My Info → Personal Details 2. Enter License Expiry Date as `18-10-2023` (past date) 3. Click Save |
| **Expected** | Warning message: "License expiry date is in the past" |
| **Actual** | Expired date is saved silently — no alert or warning shown |
| **Suspected Cause** | No date validation logic to compare license expiry date against current date before saving |

---

## 🔍 Root Cause Analysis — Bug #2: XSS Vulnerability

### What is happening?
When a user submits a post in the Buzz section containing HTML or JavaScript tags such as `<script>alert('XSS')</script>`, the application accepts and stores the input without any filtering. The raw script tag is then displayed directly on the page for all users to see.

### Why is it happening?
The root cause is the complete absence of input sanitization and output encoding. The backend API does not validate or strip HTML/JavaScript from the post content before saving it to the database. Similarly, the frontend renders the stored content directly into the DOM without escaping special characters like `<`, `>`, and `&`.

### What is the risk?
This is a Critical severity issue. In a real production environment, an attacker could inject malicious JavaScript that executes in other users' browsers — potentially stealing session cookies, redirecting users to phishing sites, or performing actions on behalf of logged-in users without their knowledge.

### How to fix it?
The fix requires two layers of protection:

**1. Server-side:** Sanitize all user input using a library such as DOMPurify (JavaScript) or bleach (Python) before storing in the database. Strip or reject any HTML tags in post content fields.

**2. Client-side:** Always escape output when rendering user-generated content. Use `textContent` instead of `innerHTML` in JavaScript, or use a templating engine that auto-escapes by default (React, Angular, Jinja2).

Both layers are needed — relying on only one is insufficient.

---

### 📄 Download Full PDF Report
[➡️ Click here to download Task1_QA_Report_MaheepGhanshani.pdf](Task1_QA_Report_MaheepGhanshani.pdf)

---

# ✅ TASK 2 — n8n API Integration Workflow

## Workflow Name
**Crypto Morning Brief** — Automated hourly crypto digest with Discord alerts

## Workflow Diagram
```
[Schedule Trigger - Every 1 Hour]
            ↓
[HTTP Request 1 → CoinGecko Markets API]
     Fetch Top 10 Cryptocurrencies
            ↓
[Code Node → Filter & Transform]
     Keep only Top 5 coins
     Extract: name, price, 24h change, market cap
            ↓
[HTTP Request 2 → CoinGecko Simple Price API]
     Enrich with real-time USD price + 24h change
            ↓
[IF Node → Bitcoin 24h change < -2%?]
        ↓ YES                    ↓ NO
[HTTP Request 3]          [No Operation]
 POST to Discord            Do nothing
 Webhook Alert
```

---

## APIs Used

| API | Endpoint | Purpose | Why Chosen |
|-----|----------|---------|------------|
| CoinGecko Markets | `/api/v3/coins/markets` | Fetch top 10 coins | Free, no API key, reliable |
| CoinGecko Simple Price | `/api/v3/simple/price` | Enrich with live prices | Single request for all coins |
| Discord Webhook | `discord.com/api/webhooks/...` | Send alert notification | Free, instant, no auth needed |

## Why CoinGecko?
- 100% free — no API key or account required
- Always returns live market data
- Comprehensive data: price, volume, market cap, 24h change
- High reliability and uptime

## Transformation Logic (Code Node)
```javascript
const coins = $input.all();
const top5 = coins.slice(0, 5);

return top5.map(item => ({
  json: {
    id: item.json.id,
    name: item.json.name,
    symbol: item.json.symbol,
    current_price: item.json.current_price,
    price_change_percentage_24h: item.json.price_change_percentage_24h,
    market_cap: item.json.market_cap,
    total_volume: item.json.total_volume
  }
}));
```
**What it does:** Filters 10 coins to top 5, removes unnecessary fields, keeps only what we need for the digest.

## Conditional Branch
- **Threshold:** Bitcoin 24h price change **< -2%**
- **True Branch → Alert:** Market is dropping — send Discord notification
- **False Branch → No Operation:** Market is stable — no action needed

## Error Handling
| Method | Where Applied |
|--------|--------------|
| Continue on Fail | All HTTP Request nodes |
| Retry on Fail | HTTP Request1 (2nd API call) |
| No Operation node | False branch — graceful handling |

**Result:** If CoinGecko API is down or returns an error — workflow logs it and continues without crashing silently.

## Sample Output (Discord)
```
🚨 Crypto Alert! Bitcoin 24h Change: -2.818% | Price: $71451
```

---

## 📸 Screenshots

### Workflow Canvas
![n8n Workflow Canvas](screenshots/canvas.png)

### Successful Execution — Discord Output
![Discord Execution](screenshots/discord_execution.png)

---

## 🚀 How to Run This Workflow

```bash
# Step 1 - Install n8n
npm install -g n8n

# Step 2 - Start n8n
n8n start

# Step 3 - Open browser
# Go to http://localhost:5678
```

1. In n8n — click **"Add workflow"**
2. Click **"..."** → **"Import from file"**
3. Select `Task2_Workflow_MaheepGhanshani.json`
4. Update Discord Webhook URL in `HTTP Request2` node
5. Click **"Execute Workflow"** to test

---

## 🛠️ Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| n8n | Latest | Workflow automation |
| Node.js | v24 | Runtime for n8n |
| CoinGecko API | v3 | Crypto market data |
| Discord Webhook | - | Notification delivery |
| JavaScript | ES6+ | Data transformation |

---

## 📞 Contact

**Name:** Maheep Ghanshani
**Email:** ghanshanimaheep04@gmail.com
