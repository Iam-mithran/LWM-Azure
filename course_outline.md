# LearnWithMithran — Azure Cloud & Azure DevOps Course Outline
> Master reference file for Claude Code sessions. Read this file at the start of every session.

---

## Channel & Course Context

- **Channel:** LearnWithMithran (YouTube)
- **Course:** Azure Cloud + Azure DevOps — A Complete Beginner's Guide
- **Total Videos:** 29 (Day 0 through Day 28)
- **Total Phases:** 9 (Phase 0 through Phase 8)
- **Max Video Length:** 2 hours per video
- **Target Audience:** Complete beginners to cloud — no prior AWS, GCP, or on-premise experience assumed. Also suitable for viewers with some prior cloud background.
- **Instructor Account:** Paid Azure account (used during teaching to demonstrate all features including paid-only ones)
- **Student Account:** Azure Free Tier (assumed for most viewers; some may choose paid for practice)

---

## Teaching Philosophy & Ground Rules

1. **Azure-Native Teaching** — Explain every concept from first principles as if the viewer has never used any cloud platform. No comparisons to AWS, GCP, or on-premise.
2. **Console-First Always** — Every concept is demonstrated via the Azure Portal (GUI). Students learn by clicking, not typing commands.
3. **CLI is Contextual Only** — Azure CLI is shown only where it genuinely saves time (e.g., Bicep/Terraform deploys). CLI usage is capped at 3–5% of total content. No standalone CLI-only video.
4. **Hands-On Every Day** — Every video includes at least one practical demo or lab the viewer can follow along.
5. **No Assumed Cloud Knowledge** — Treat every viewer as starting from zero. Define all terms clearly.
6. **Account Tier Transparency** — The instructor teaches on a paid Azure account. Students are assumed to be on the Azure Free Tier. Each day's Hands-On Demo must clearly label every step:
   - ✅ **Free Tier** — students can follow along
   - 💳 **Paid (Instructor Demo)** — instructor demonstrates; students should watch, skip, or activate a free trial to practice
7. **Script Structure Per Video:**
   - Hook (what problem does this solve?) — ~2 min
   - Core concept explanation — ~10–15 min
   - Portal demo / hands-on — ~20–40 min
   - Summary + what's next — ~2–3 min

---

## Phase Overview

| Phase | Days | Topic |
|-------|------|-------|
| Phase 0 | Day 0 – Day 1 | Foundations + Account Setup |
| Phase 1 | Day 2 – Day 5 | Compute |
| Phase 2 | Day 6 – Day 8 | Networking |
| Phase 3 | Day 9 – Day 12 | Storage + Databases |
| Phase 4 | Day 13 – Day 16 | Identity, Security + Monitoring |
| Phase 5 | Day 17 – Day 21 | Azure DevOps |
| Phase 6 | Day 22 – Day 23 | Infrastructure as Code (IaC) |
| Phase 7 | Day 24 – Day 26 | Containers + AKS |
| Phase 8 | Day 27 – Day 28 | Capstone Project |

Individual day content lives in the `days/` folder — e.g., `days/day05_functions.md`.

---

## Instructions for Claude Code Sessions

When starting a new session to write course content:

1. **Read this file first** to load course context.
2. **Read the specific day file** from `days/` for the day you are working on (e.g., `days/day05_functions.md`).
3. **Output format for each day's script:**
   - File name: `dayXX_topic_name.md` (e.g., `day05_azure_functions.md`)
   - Sections: Hook → Core Concepts → Portal Demo Steps → Summary → Key Takeaways
   - All Portal Demo steps must carry ✅ or 💳 tier labels
4. **No AWS, GCP, or on-premise comparisons** — Azure-native explanations only.
5. **Scope per session:** Write ONE day at a time.
6. **Length target:** 1,500–3,000 words per day (maps to ~30–60 min of narration; demos fill remaining video time).
7. **Tone:** Conversational, enthusiastic, instructor-speaking-to-camera. Not dry documentation.

---

*Last updated: May 2026 | LearnWithMithran | Azure Cloud + Azure DevOps Course*
