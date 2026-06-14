# AdvisorAI — Claude Code Project Brief

> Hand this file to Claude Code and say: "Build this project step by step, starting with Stage 1."

---

## Project Overview

**AdvisorAI** is a mini contact centre intelligence dashboard that simulates what a real Advisor Desktop does — summarising calls, detecting intent, surfacing Next Best Actions, and tracking trends over time.

**Stack:** Vanilla HTML + CSS + JavaScript (single file per stage) + Anthropic Claude API  
**No frameworks. No database. No backend.** Everything runs in the browser.  
**Skill level:** Beginner-friendly. Each stage builds on the last.

---

## What You're Learning (Mapped to Real PM Experience)

| What You Build | Real-World Equivalent |
|---|---|
| Claude API call with structured prompt | GenAI call summarisation pipeline |
| JSON output parsing + display | Structured data extraction from LLM |
| Intent classification | Customer intent detection system |
| NBA recommendation logic | Next Best Action engine |
| Injecting history into prompt | RAG (Retrieval Augmented Generation) |
| Sentiment scoring | NPS proxy / post-call CSAT |
| localStorage persistence | Interaction History platform |
| Metrics dashboard | Real-time analytics for ops leaders |
| Guardrail validation | Hallucination detection & mitigation |
| Loading states & error handling | Production-grade UX |

---

## Project Folder Structure

```
advisorai/
├── index.html        ← Stage 1: Core analyser
├── dashboard.html    ← Stage 2: Call history & log
├── profile.html      ← Stage 3: Customer profile + history injection
├── metrics.html      ← Stage 4: Charts & trend dashboard
└── BRIEF.md          ← This file
```

---

---

# STAGE 1 — The Core Analyser

**Goal:** Paste a transcript → get AI analysis back in a clean card layout.  
**File:** `index.html`  
**Time:** ~2 hours  
**Teaches:** Prompt engineering, API calls, JSON parsing, structured outputs, error handling

---

## What Stage 1 Builds

A single-page app with:
- A large textarea for pasting a call transcript
- A "Analyse Call" button
- A results panel showing: Summary, Intent badge, Sentiment badge, NBA recommendation, Resolution status
- A loading state while the API call is in flight
- Error handling if the API fails or returns malformed JSON

---

## Stage 1 — Full Specification

### Layout (top to bottom)

```
┌─────────────────────────────────────────────────┐
│  🎧 AdvisorAI          Contact Centre Intel Tool │
├─────────────────────────────────────────────────┤
│                                                 │
│  PASTE CALL TRANSCRIPT                          │
│  ┌─────────────────────────────────────────┐    │
│  │                                         │    │
│  │  [large textarea — 10 rows]             │    │
│  │                                         │    │
│  └─────────────────────────────────────────┘    │
│                                                 │
│  Customer ID (optional): [____________]         │
│                                                 │
│  [ Analyse Call ▶ ]                             │
│                                                 │
├─────────────────────────────────────────────────┤
│  ANALYSIS RESULTS                               │
│                                                 │
│  ┌──────────────────────────────────────────┐   │
│  │ 📋 SUMMARY                               │   │
│  │ [2-3 sentence summary text]              │   │
│  └──────────────────────────────────────────┘   │
│                                                 │
│  [INTENT: Booking] [SENTIMENT: Positive]        │
│  [RESOLVED: ✓ Yes]                              │
│                                                 │
│  ┌──────────────────────────────────────────┐   │
│  │ ⚡ NEXT BEST ACTION                       │   │
│  │ [1-sentence recommendation]              │   │
│  └──────────────────────────────────────────┘   │
│                                                 │
│  ┌──────────────────────────────────────────┐   │
│  │ 🛡️ CONFIDENCE                             │   │
│  │ [confidence score + any warnings]        │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

### The Claude API Call

Use the Anthropic API directly from the browser using `fetch`.

**Endpoint:** `https://api.anthropic.com/v1/messages`  
**Model:** `claude-sonnet-4-6`  
**API Key handling:** Prompt the user to enter their API key in a settings panel at the top of the page. Store it in `localStorage` as `advisorai_api_key`. Never hardcode it.

**The System Prompt (copy this exactly):**

```
You are an AI assistant that analyses customer service call transcripts for a 
travel and financial services contact centre. 

You MUST respond with valid JSON only. No preamble, no explanation, no markdown 
code fences. Just the raw JSON object.

Your response must match this exact schema:
{
  "summary": "string — 2 to 3 sentences summarising what happened on the call",
  "intent": "string — exactly one of: booking | cancellation | complaint | upsell | general | refund",
  "sentiment": "string — exactly one of: positive | neutral | negative",
  "resolved": boolean,
  "resolution_notes": "string — one sentence on what was resolved or what remains open",
  "next_best_action": "string — one sentence recommendation for what the advisor should do next or follow up on",
  "key_topics": ["array", "of", "up", "to", "5", "topic", "strings"],
  "confidence": number between 0 and 1,
  "warning": "string or null — flag if transcript is too short, unclear, or if you are uncertain about any field"
}

Rules:
- If the transcript is fewer than 50 words, set confidence below 0.5 and add a warning.
- If you cannot determine intent clearly, set intent to general.
- resolved must be true only if the customer's primary issue was fully addressed.
- Never invent information not present in the transcript.
```

**The User Message:**

```
Analyse this call transcript:

---
{TRANSCRIPT_TEXT}
---
```

---

### Parsing & Guardrails

After receiving the API response:

1. Extract the text content from `response.content[0].text`
2. Attempt `JSON.parse()` on the raw text
3. If parsing fails, try stripping markdown fences (` ```json ` and ` ``` `) then parse again
4. If still fails, show an error: "The AI returned an unexpected format. Please try again."
5. Validate that all required fields exist before rendering
6. If `warning` is not null, display it in a yellow warning banner above the results

This teaches you: **why guardrails exist in production AI systems**

---

### Colour Scheme & Design

```css
/* Design tokens */
--bg-primary:    #0F1117;   /* near-black background */
--bg-surface:    #1A1D27;   /* card backgrounds */
--bg-elevated:   #222536;   /* hover states */
--border:        #2E3250;   /* subtle borders */
--accent:        #5B6AF0;   /* primary blue-purple */
--accent-soft:   #5B6AF020; /* tint */
--text-primary:  #E8E9F0;   /* main text */
--text-secondary:#8B8FA8;   /* labels and metadata */
--success:       #34D399;   /* positive / resolved */
--warning:       #FBBF24;   /* warnings */
--error:         #F87171;   /* errors / negative */
--font:          'Inter', sans-serif;
```

**Intent badge colours:**
- booking → blue (`#3B82F6`)
- cancellation → red (`#EF4444`)
- complaint → orange (`#F97316`)
- upsell → green (`#10B981`)
- refund → yellow (`#EAB308`)
- general → gray (`#6B7280`)

**Sentiment badge colours:**
- positive → green
- neutral → gray
- negative → red

---

### Loading State

While the API call is in flight:
- Disable the Analyse button and change its text to "Analysing..."
- Show a pulsing skeleton loader in the results area (three animated grey bars)
- This teaches you: **why latency matters in real-time agent assist tools**

---

### Save to History

After a successful analysis, automatically save a record to `localStorage`:

```javascript
const record = {
  id: Date.now(),
  timestamp: new Date().toISOString(),
  customer_id: customerId || "Unknown",
  transcript_snippet: transcript.slice(0, 150) + "...",
  ...analysisResult  // spread all fields from the Claude response
};

const history = JSON.parse(localStorage.getItem('advisorai_history') || '[]');
history.unshift(record); // newest first
localStorage.setItem('advisorai_history', JSON.stringify(history));
```

---

### Sample Transcripts to Test With

Include these as clickable "Load Sample" buttons so you can test without typing:

**Sample 1 — Booking (Positive)**
```
Agent: Thank you for calling Chase Travel, my name is Sarah. How can I help you today?
Customer: Hi Sarah, I'd like to book a round trip from New York to London in business class for 
two passengers in March.
Agent: Absolutely, I'd be happy to help with that. Do you have specific dates in mind?
Customer: We're flexible between March 10th and March 20th. We'd prefer British Airways if 
possible and we'd like to use our points to offset the cost.
Agent: Great choice. I can see you have 180,000 Chase Ultimate Rewards points which would 
cover about $1,800 off the fare. I'm finding availability on March 12th returning March 19th 
on British Airways. The total would be $3,200 per person minus the $1,800 credit, so $4,600 
for both. Shall I proceed?
Customer: That sounds perfect, yes please go ahead.
Agent: Wonderful, I've confirmed your booking. You'll receive a confirmation email shortly. 
Is there anything else I can help you with?
Customer: No that's everything, thank you so much!
Agent: Have a wonderful trip to London!
```

**Sample 2 — Complaint (Negative)**
```
Agent: Chase Travel, this is Mike speaking. How can I assist you?
Customer: I am absolutely furious right now. I booked a flight two months ago and I just 
found out it was cancelled. Nobody contacted me, I found out by checking my email randomly.
Agent: I sincerely apologise for that experience. Let me pull up your booking right away. 
Can I get your booking reference?
Customer: It's CHT-88423. I have a conference I need to attend. This is completely 
unacceptable. I need to be in Chicago by Monday morning no matter what.
Agent: I can see the cancellation — the airline cancelled this flight due to operational 
reasons. I'm very sorry you weren't notified proactively. I have a few options for you: 
I can rebook you on an alternative flight Sunday evening arriving 9pm, or Monday morning 
arriving 7am, or I can process a full refund.
Customer: The Sunday evening flight works. But I want compensation for this. I nearly 
missed finding out entirely.
Agent: Understood. I'm rebooking you now on the Sunday flight and I'll apply a $150 travel 
credit to your account as an apology. You'll get a new confirmation in the next few minutes.
Customer: Fine. But I'm not happy about this.
```

**Sample 3 — Upsell Opportunity**
```
Agent: Good afternoon, Chase Travel, you're speaking with James.
Customer: Hi, I need to book a one-way ticket to Miami for next Friday.
Agent: Of course. Any preference on airline or time of day?
Customer: Cheapest option is fine. I usually just go economy.
Agent: I have a Spirit Airlines flight at 6am for $89, or a Delta flight at 11am for $134. 
The Delta flight includes a carry-on bag which the Spirit fare doesn't.
Customer: I'll go with Spirit I guess. Oh wait, I do have a carry-on.
Agent: In that case the Spirit fare would actually be $89 plus $65 for the bag, so $154 
total versus Delta at $134 all-in. Delta would actually be cheaper for you.
Customer: Oh wow I didn't realise. Yeah let's do Delta then.
Agent: Smart choice. And I can see you have 23,000 points — this $134 fare is eligible for 
a points boost promotion this week, you'd earn triple points on this booking.
Customer: Oh nice, yes definitely book it with that promotion.
Agent: All confirmed! Anything else I can help with today?
Customer: No that's great, thanks for the tip about the bag fee!
```

---

---

# STAGE 2 — Call History Dashboard

**Goal:** See all past analysed calls in a searchable, filterable table.  
**File:** `dashboard.html`  
**Time:** ~1.5 hours  
**Teaches:** Data persistence, filtering, aggregation, reading your own stored data

---

## What Stage 2 Builds

A dashboard page with:
- A summary stats row at the top (total calls, % resolved, most common intent, average sentiment score)
- A filterable table of all past calls from localStorage
- Filter by: intent, sentiment, resolved status, date range
- Click any row to expand and see the full analysis
- A "Clear History" button (with confirmation dialog)
- A nav bar linking to all four pages

---

## Stage 2 — Full Specification

### Summary Stats Row

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ 47       │  │ 78%      │  │ Booking  │  │ Positive │
│ Total    │  │ Resolved │  │ Top      │  │ Avg      │
│ Calls    │  │          │  │ Intent   │  │ Sentiment│
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

Calculate these from all records in `advisorai_history` localStorage.

For average sentiment, convert to a score first:
- positive = 1
- neutral = 0
- negative = -1

Then average, and display as: Positive (>0.3), Mixed (−0.3 to 0.3), Negative (<−0.3)

---

### Calls Table

Columns:
| # | Time | Customer ID | Intent | Sentiment | Resolved | NBA Preview | Actions |
|---|------|-------------|--------|-----------|----------|-------------|---------|

- Sort by time descending (newest first) by default
- Click column headers to sort
- "Actions" column: 👁 View | 🗑 Delete

**Expanded row** (click 👁 to toggle):
Shows full summary, all key topics as tags, resolution notes, and full NBA recommendation.

---

### Filter Bar

```
[Intent ▼] [Sentiment ▼] [Resolved ▼] [Search customer ID...] [Date from] [Date to] [Reset]
```

Filters are applied client-side in JavaScript — no server needed.

---

### Nav Bar

Add a consistent top navigation to both `index.html` and `dashboard.html`:

```
🎧 AdvisorAI  |  [Analyse Call]  [Dashboard]  [Customer Profiles]  [Metrics]
```

---

---

# STAGE 3 — Customer Profiles & History Injection

**Goal:** Remember repeat customers and inject their history into the AI prompt.  
**File:** `profile.html` + updates to `index.html`  
**Time:** ~2 hours  
**Teaches:** RAG (Retrieval Augmented Generation), prompt context injection, personalisation

---

## What Stage 3 Builds

- A customer profile page: enter a Customer ID to see all their past interactions
- A customer profile summary card (how many calls, most common intent, satisfaction trend)
- **Most importantly:** When analysing a new call in `index.html`, if a Customer ID is entered, automatically fetch that customer's last 3 interactions from localStorage and inject them into the Claude prompt

This is **RAG in practice** — you are retrieving relevant data and injecting it into the prompt context so the AI can generate a better, more personalised response.

---

## Stage 3 — Full Specification

### Profile Page Layout

```
┌─────────────────────────────────────────┐
│  👤 Customer Profile                    │
│  Customer ID: [__________] [Search]     │
├─────────────────────────────────────────┤
│  CUSTOMER SUMMARY                       │
│  ID: CUST-1042                          │
│  Total Calls: 6                         │
│  Last Contact: 3 days ago               │
│  Most Common Intent: Booking            │
│  Satisfaction Trend: Improving ↑        │
├─────────────────────────────────────────┤
│  INTERACTION HISTORY                    │
│  [Timeline of past calls, newest first] │
│  Each shows: date, intent, sentiment,   │
│  summary snippet, NBA from that call    │
└─────────────────────────────────────────┘
```

---

### History Injection (the RAG piece)

Update the `index.html` Analyse Call flow:

**Before building the user message for Claude, check if a Customer ID was entered:**

```javascript
function buildPrompt(transcript, customerId) {
  let contextBlock = "";
  
  if (customerId) {
    const history = JSON.parse(localStorage.getItem('advisorai_history') || '[]');
    const customerHistory = history
      .filter(r => r.customer_id === customerId)
      .slice(0, 3); // last 3 interactions only
    
    if (customerHistory.length > 0) {
      contextBlock = `
CUSTOMER HISTORY CONTEXT (last ${customerHistory.length} interactions):
${customerHistory.map((r, i) => `
Interaction ${i + 1} — ${new Date(r.timestamp).toLocaleDateString()}:
- Intent: ${r.intent}
- Sentiment: ${r.sentiment}  
- Summary: ${r.summary}
- Was Resolved: ${r.resolved}
- NBA at the time: ${r.next_best_action}
`).join('')}

Use this context to:
1. Note if this customer is a repeat caller with unresolved issues
2. Personalise the Next Best Action based on their history
3. Flag in your warning field if this customer has had 2+ negative interactions

---
`;
    }
  }
  
  return `${contextBlock}Analyse this call transcript:\n\n---\n${transcript}\n---`;
}
```

**Why this matters:** This is exactly what your JPMC Interaction History platform does. You're injecting past context into the AI's reasoning. The AI can now say "this customer called about the same booking issue last week and it wasn't resolved — prioritise resolution."

---

### What to Observe

After building Stage 3, run these experiments:
1. Analyse a complaint call with no customer history → note the generic NBA
2. Save that call under Customer ID "CUST-001"
3. Analyse another complaint call for the same customer → notice how the NBA changes when history is injected
4. This is the "aha moment" for understanding RAG

---

---

# STAGE 4 — Metrics Dashboard

**Goal:** Visualise trends across all analysed calls.  
**File:** `metrics.html`  
**Time:** ~1.5 hours  
**Teaches:** Data aggregation, business metrics, analytics thinking, charting

---

## What Stage 4 Builds

A metrics page with:
- Intent breakdown (pie or bar chart)
- Sentiment trend over time (line chart)
- Resolution rate trend over time
- Top NBA themes (word frequency from all NBA recommendations)
- A "simulate NPS" score based on sentiment data

---

## Stage 4 — Full Specification

### Use Chart.js (CDN)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js"></script>
```

No npm, no build step. Just include the CDN link.

---

### Chart 1 — Intent Breakdown (Doughnut Chart)

Count occurrences of each intent across all history records.

```javascript
const intentCounts = history.reduce((acc, r) => {
  acc[r.intent] = (acc[r.intent] || 0) + 1;
  return acc;
}, {});
```

Display as a doughnut chart with the intent-colour mapping from Stage 1.

---

### Chart 2 — Sentiment Trend (Line Chart)

Group records by day. For each day, calculate average sentiment score (positive=1, neutral=0, negative=-1).

Display as a line chart with days on X axis and score on Y axis (-1 to 1).

Add a dashed reference line at 0 (neutral baseline).

---

### Chart 3 — Resolution Rate (Bar Chart)

Group records by week. For each week, show % resolved as a bar.

Target line at 75% (industry benchmark for FCR).

---

### Simulated NPS Score

Calculate from your sentiment data:

```javascript
function simulateNPS(history) {
  if (history.length === 0) return null;
  
  // Map sentiment to NPS proxy
  // positive = promoter (score 9-10)
  // neutral = passive (score 7-8)  
  // negative = detractor (score 0-6)
  
  const promoters = history.filter(r => r.sentiment === 'positive').length;
  const detractors = history.filter(r => r.sentiment === 'negative').length;
  const total = history.length;
  
  const nps = Math.round(((promoters - detractors) / total) * 100);
  return nps; // Range: -100 to +100
}
```

Display as a large number with context:
- Above 50: Excellent 🟢
- 20–50: Good 🟡
- 0–20: Needs improvement 🟠
- Below 0: Critical 🔴

**Why this matters:** This is how your JPMC team tracked NPS improvement from the AI initiatives. You're building the same mental model.

---

### NBA Theme Analysis

Extract all `next_best_action` strings from history. Split into words, remove stop words, count frequency.

Display as a simple ranked list:
```
Top NBA Themes:
1. "follow-up" — mentioned 12 times
2. "travel credit" — mentioned 8 times
3. "rebooking" — mentioned 6 times
```

This simulates how a product team would analyse what the AI is recommending to find patterns.

---

---

# BONUS CHALLENGES (After Stage 4)

Once all 4 stages are complete, try these extensions:

---

## Bonus 1 — Guardrail Validator

Add a second Claude API call after the main analysis that acts as a "reviewer":

**Prompt:** `"Review this JSON analysis of a call transcript. Check if the summary contains any information NOT present in the transcript. If it does, flag it as a hallucination. Return JSON: {hallucination_detected: boolean, flagged_claims: [array of strings], verdict: 'safe' | 'review_needed'}."`

This teaches you: **how production AI systems validate their own outputs.**

---

## Bonus 2 — Real-Time Typing Analysis

Instead of analysing after the transcript is pasted, add a "Live Mode" where analysis triggers as the user types (debounced — wait 2 seconds after the last keypress).

Show intent and sentiment updating in real-time as more of the transcript is entered.

This teaches you: **why real-time agent assist is technically harder than batch processing.**

---

## Bonus 3 — Routing Recommendation

Add a "Routing" panel to the analysis results:

Based on intent + sentiment + customer history, recommend which advisor tier should handle this call:
- Tier 1 (Standard): general, booking, positive/neutral
- Tier 2 (Senior): complaint, negative, or repeat caller
- Tier 3 (Specialist): high-value customer, complex upsell, unresolved from previous call

This teaches you: **the logic behind intelligent call routing.**

---

## Bonus 4 — Export & Reporting

Add an "Export" button to the dashboard that generates a CSV of all call history.

Add a "Weekly Report" button that sends all this week's data to Claude and asks it to write a 3-paragraph executive summary of contact centre performance.

This teaches you: **how ops leaders use AI to generate insights from data.**

---

---

# LEARNING REFLECTION QUESTIONS

After each stage, ask yourself these questions. They're the same questions a PM would ask in a product review:

**After Stage 1:**
- What happens when the transcript is ambiguous? How does the AI handle it?
- What would break this in production? (Very long transcripts? Non-English? Background noise in transcript?)
- Why does the system prompt matter so much? Try removing it and see what happens.

**After Stage 2:**
- If you were an ops manager, what would you look at first in this dashboard?
- What's missing that would make this more useful? (That's a real product discovery question.)
- What does "resolution rate" actually measure? Is it a good proxy for quality?

**After Stage 3:**
- How did the NBA change when customer history was injected? Was it better?
- What's the risk of injecting too much history? (Token limits, relevance degradation)
- This is RAG — what would a database-backed version of this look like at scale?

**After Stage 4:**
- Is your simulated NPS a fair proxy? What does it miss?
- If the sentiment trend was declining, what would you hypothesise first?
- What decisions would you make differently if the top NBA theme was "refund" vs "upsell"?

---

---

# FIRST CLAUDE CODE SESSION — STARTER PROMPT

Copy and paste this to start your first Claude Code session:

```
I want to build a project called AdvisorAI — a mini contact centre intelligence 
dashboard. I have a detailed project brief in BRIEF.md in this folder. 

Please start with Stage 1 only (the Core Analyser). Build it as a single 
self-contained index.html file with all CSS and JavaScript inline.

Key requirements for Stage 1:
- API key entry field stored in localStorage (key: advisorai_api_key)
- Large textarea for transcript input  
- "Load Sample" buttons for 3 sample transcripts (included in the brief)
- Anthropic API call to claude-sonnet-4-6 using the system prompt in the brief
- Parse the JSON response with fallback error handling
- Display results in cards: Summary, Intent badge, Sentiment badge, Resolved status, 
  NBA recommendation, Confidence score, Warning banner if warning is not null
- Loading skeleton while API call is in flight
- Auto-save result to localStorage key 'advisorai_history' after each analysis
- Dark colour scheme using the design tokens in the brief

After you build it, explain what each part of the Claude API call does so I 
understand the code, not just have it.
```

---

*Built to accompany the AdvisorAI interview prep project. Each stage maps directly to concepts on Rajul Mishra's CV.*
