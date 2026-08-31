<div align="center">

# MemoRx

**Pharmacology board prep that adapts to what you keep forgetting.**

*A NAPLEX study app for pharmacy students — one drug a day, quizzed until it sticks.*

![Platform](https://img.shields.io/badge/platform-iOS%2017%2B-C9A46A?style=for-the-badge&labelColor=0a0a0a)
![Swift](https://img.shields.io/badge/SwiftUI-Swift%206-D9B87C?style=for-the-badge&labelColor=0a0a0a)
![Backend](https://img.shields.io/badge/backend-Supabase-8a8f98?style=for-the-badge&labelColor=0a0a0a)
![Dataset](https://img.shields.io/badge/dataset-201%20drugs-C9A46A?style=for-the-badge&labelColor=0a0a0a)
![Status](https://img.shields.io/badge/status-shipped-6b8f71?style=for-the-badge&labelColor=0a0a0a)

</div>

---

## The idea

Pharmacy students do not fail boards because they never saw a drug. They fail because they saw it in September and the exam is in May.

MemoRx is built around that gap. Instead of a flashcard pile you grind until it blurs, it holds one curated drug per day for everyone, quizzes you on it, and then keeps quietly re-surfacing the drugs you got shaky on — right before you would have lost them. The dataset is hand-built rather than scraped, because a wrong mechanism of action in a study app is worse than no study app.

> Same drug, same day, for everyone. It makes the leaderboard fair and it makes the studying social.

---

## Screenshots

<div align="center">

<img src="screenshots/01-daily-drug.png" width="32%" alt="Daily drug card — today's drug with a quick quiz" />
<img src="screenshots/02-daily-study.png" width="32%" alt="Daily study view — streak tracking and weekly progress" />
<img src="screenshots/03-adaptive-quiz.png" width="32%" alt="Adaptive quiz — answer feedback with mechanism explanation" />

<sub>Daily drug · Streaks & daily study · Adaptive quizzing</sub>

</div>

---

## What's inside

<table>
<tr><td width="30%"><b>201 drugs</b></td><td>Hand-curated, not scraped. Each one carries 16 structured fields.</td></tr>
<tr><td><b>21 collections</b></td><td>Therapeutic areas — pain &amp; fever, cardiovascular, endocrine, infectious disease, psych, and more.</td></tr>
<tr><td><b>104 sub-collections</b></td><td>The level where real distinctions live: non-opioid analgesics vs. opioids, not just "pain."</td></tr>
<tr><td><b>16 fields per drug</b></td><td>Mechanism, indications, contraindications, interactions, side effects, warnings, monitoring, dosage, counseling points, and clinical pearls.</td></tr>
<tr><td><b>82 Swift files</b></td><td>~24,700 lines. All SwiftUI, no UIKit bridge layer.</td></tr>
</table>

---

## How the quizzing works

The engine treats a drug as something you have a decaying grip on, not a card you have or haven't flipped.

**Spaced repetition.** An SM-2 inspired scheduler tracks per-drug mastery and surfaces each drug just before the point it would be forgotten. Rating a drug as hard pulls it forward; getting it right repeatedly pushes it out.

**Two question altitudes.** Roughly 70% of questions are drug-specific ("what is the mechanism of amoxicillin?") and 30% are class-level ("which of these is *not* a beta-lactam?"). Drug-specific questions build recall; class-level questions build the comparative reasoning boards actually test.

**No repeats.** Questions are generated across the drug's structured fields and shuffled per attempt, so a second pass through the same drug is a genuinely different quiz rather than a memorized answer position.

**Difficulty is yours, not global.** Each user carries their own difficulty rating per drug, which feeds back into scheduling weight.

---

## Architecture

```mermaid
flowchart TB
    subgraph CLIENT["📱 iOS — SwiftUI"]
        UI["Views<br/><i>quiz · drug cards<br/>progress · leaderboard</i>"]
        ENG["Engines<br/><i>quiz generation<br/>spaced repetition<br/>exam prep planner</i>"]
        CACHE["Offline cache<br/><i>bundled JSON fallback<br/>local progress state</i>"]
    end

    subgraph SERVER["☁️ Supabase"]
        DB["Postgres<br/><i>content · progress<br/>leaderboard</i>"]
        AUTH["Auth<br/><i>Sign in with Apple<br/>anonymous sessions</i>"]
        FN["Edge Functions<br/><i>Deno / TypeScript</i>"]
        CRON["Scheduled jobs<br/><i>daily challenge<br/>weekly archival</i>"]
    end

    STORE["🍎 StoreKit 2<br/><i>subscriptions</i>"]

    UI --> ENG --> CACHE
    CACHE <--> DB
    UI --> AUTH
    STORE --> FN --> DB
    CRON --> DB

    style CLIENT fill:#0a0a0a,stroke:#C9A46A,stroke-width:2px,color:#F5F0E6
    style SERVER fill:#0a0a0a,stroke:#8a8f98,stroke-width:2px,color:#F5F0E6
    style UI fill:#1a1614,stroke:#C9A46A,color:#F5F0E6
    style ENG fill:#241d17,stroke:#D9B87C,stroke-width:2px,color:#F5F0E6
    style CACHE fill:#1a1614,stroke:#8a8f98,color:#F5F0E6
    style DB fill:#141719,stroke:#8a8f98,color:#F5F0E6
    style AUTH fill:#141719,stroke:#8a8f98,color:#F5F0E6
    style FN fill:#141719,stroke:#8a8f98,color:#F5F0E6
    style CRON fill:#141719,stroke:#8a8f98,color:#F5F0E6
    style STORE fill:#1a1614,stroke:#C9A46A,color:#F5F0E6
```

**Cloud-first, offline-tolerant.** The app asks Supabase for content on launch. If the network fails or returns empty, it falls back to the last cached copy, then to JSON bundled in the binary — so a student on a hospital floor with no signal still gets their quiz.

**Server-authoritative progress.** Streaks, XP, and milestones are computed server-side rather than trusted from the device. A client that lies about its streak does not get a streak.

**Receipts are verified, not decoded.** Subscription state is validated server-side against Apple's signed payloads with full certificate-chain verification anchored to Apple's root CA — not by reading the claims and hoping.

---

## Design principles

<table>
<tr><td width="34%"><b>🩺 Correctness over volume</b></td><td>201 drugs a pharmacist would sign off on beats 2,000 scraped ones. Every field is reviewed; wrong pharmacology in a study app is actively harmful.</td></tr>
<tr><td><b>🕯 One drug a day</b></td><td>The unit of study is a day, not a session. Small, finishable, repeatable — the schedule does the work so motivation does not have to.</td></tr>
<tr><td><b>📵 Studying works offline</b></td><td>Content is cached and bundled. Signal is not a prerequisite for review.</td></tr>
<tr><td><b>🏛 Restrained by default</b></td><td>Black and gold, serif headings, generous space. A study tool should feel calm and serious, not gamified into noise — the streak counter is the loudest thing on screen and that is deliberate.</td></tr>
<tr><td><b>🔒 Health-adjacent privacy</b></td><td>No user identifiers in crash telemetry, anonymous sessions supported, account deletion is a first-class action rather than an email request.</td></tr>
<tr><td><b>⚖️ Fair competition</b></td><td>Everyone gets the same drug on the same day. The leaderboard measures consistency, not who found the easiest deck.</td></tr>
</table>

---

## What it isn't

An AI question generator · a scraped drug database · a flashcard app with a pharmacology skin · a social network · a clinical reference for practice · free forever

---

## North Star

> A student opens MemoRx for four minutes a day for eight months, and walks into the NAPLEX having genuinely retained two hundred drugs — without ever having built a study schedule themselves.

---

<div align="center">

**Source is private.** This repository is a showcase of the work, its architecture, and its design.

<sub>Built by <a href="https://github.com/ctxako">@ctxako</a></sub>

</div>
