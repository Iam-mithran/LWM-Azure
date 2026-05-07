# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the course content repository for **LearnWithMithran — Azure Cloud + Azure DevOps: A Complete Beginner's Guide** (YouTube). It is a documentation-only repo — no build tools, tests, or application code.

**Target audience:** Complete beginners to cloud — no prior AWS, GCP, or on-premise experience assumed.

**Instructor account:** Paid Azure. **Student account:** Azure Free Tier.

---

## Working on Course Content

### Session workflow

1. Read `course_outline.md` — teaching philosophy, tier-labeling rules, phase overview.
2. Read the specific day file from `days/` — e.g., `days/day05_functions.md`.
3. Work on **one day at a time**.
4. Write the output script to a new file: `dayXX_topic_name.md` (e.g., `day05_azure_functions.md`).

### Output format per day

Every day's script must follow this structure:

1. **Hook** — what problem does this solve? (~2 min)
2. **Core Concepts** — Azure-native explanation from first principles (~10–15 min)
3. **Portal Demo Steps** — every step labeled ✅ (Free Tier) or 💳 (Paid — instructor demo). **No time cap** — cover every important feature completely. Can run 40–120 min depending on scope.
4. **Summary + What's Next** (~2–3 min)
5. **Key Takeaways**

### Length and tone

- **Length target:** No fixed cap. Write as much as the topic demands to cover all key features, options, and use cases. A single day can be 3,000–6,000+ words if the topic requires it. The goal is technical completeness.
- **Depth standard:** Deep dive, not high level. Cover every important feature of each service. Do not skip steps or options for brevity. Students want thorough, hands-on technical content.
- **Tone:** Conversational and enthusiastic, instructor speaking to camera — not dry documentation

---

## Teaching Philosophy (apply to all content)

- **Azure-native only:** Explain every concept from first principles. No comparisons to AWS, GCP, or on-premise in any scripts.
- **Console-first:** Every concept demonstrated via the Azure Portal GUI. CLI only where it genuinely saves time (Bicep/Terraform deploys) — capped at 3–5% of content.
- **No assumed cloud knowledge:** Treat every viewer as starting from zero. Define all terms.
- **Hands-on every day:** Every video includes at least one practical demo the viewer can follow.
- **Tier transparency:** Label every demo step ✅ Free Tier or 💳 Paid (Instructor Demo). See `course_outline.md` for the full rule and which days have paid-only demos.

---

## Course Structure at a Glance

| Phase | Days | Topic |
|-------|------|-------|
| 0 | 0–1 | Foundations + Account Setup |
| 1 | 2–5 | Compute (VMs, App Service, Functions) |
| 2 | 6–8 | Networking (VNet, Load Balancers, VPN) |
| 3 | 9–12 | Storage + Databases |
| 4 | 13–16 | Identity, Security + Monitoring |
| 5 | 17–21 | Azure DevOps |
| 6 | 22–23 | Infrastructure as Code |
| 7 | 24–26 | Containers + AKS |
| 8 | 27–28 | Capstone Project |

Full day details (goals, key topics, demo steps) are in `days/dayXX_*.md`.
