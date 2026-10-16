# Studio 3 — Team 6 Submission

**Team members:** Bhanu Harsha Yanamadala, Vishwani Singh, Wendi Shi

**Tool:** claude.ai (web/mobile app) · **Model:** Claude Opus 5.5 · **Connectors:** PubMed, LibKey · **Runs:** 9 and 15 Oct 2026

---

## 1. The task

**Literature review screening** for: *How is machine learning used in extended reality (VR/AR/MR) systems for patients with musculoskeletal or neurological disorders, and how does it help standardize assessment or rehabilitation?* Build a PubMed search, screen titles/abstracts against inclusion criteria, check borderline full text via LibKey, and extract what the ML does.

- **Who and how long:** a graduate student or research assistant, usually with a second reviewer who resolves disagreements; a supervisor owns the criteria, and a librarian may help with the search. Days to weeks, hundreds of records over many sittings.
- **Why it needs human control:** criteria are judgment calls that shift as you read (does a stereo projector count as XR?); exclusions must be defensible; the search must be reproducible and reported (PRISMA); and a wrongly excluded paper is invisible unless the process is transparent.
- **How we want it to work:** checkpoints, not end-to-end runs — (1) agent proposes the search, human approves; (2) agent screens a batch with reasons, human corrects; (3) agent screens the next batch applying corrections and surfaces policy calls; (4) extraction with sources for spot checks.

---

## 2. Diagnosis

| Question | What we saw |
|---|---|
| **Capabilities** | Strong screening judgment: **0 violations** of our three correction rules in both runs, full-text checks when abstracts were unclear, and policy calls surfaced instead of decided silently (stereo projection #35, recording-only headset #39). Weaker: a claim from memory presented as a finding (HOLOBALANCE "is an AR-glasses platform", no full text), an inference treated as evidence (#34 "same group used a Vive"), and ML roles summarized as the paper claims rather than how well they work (#27, R² = 0.014–0.216). |
| **Tools** | PubMed + LibKey were enough, but PubMed **did not reliably apply the 2015 date limit** (leaks in four sessions), allows **at most 20 Boolean operators and no wildcards** (neither Part 5 agent knew the operator limit; Chat A also used wildcards), and connectors had to be switched on in each new chat (Figure 1). |
| **Capability gap** | Search strategy and reproducibility: same prompt → two search strings → 149 vs 51 hits → 2 vs 10 includes. No persistent record of decisions across sessions. |
| **Interface** | The main problem. The search — the decision that matters most — was never shown for approval, while harmless read-only lookups triggered permission prompts. In Run 2 the agent **ran and discarded a 285-hit search without telling us**; we only found out by asking. |

![New screening chat — agent asks for PubMed and LibKey connectors to be turned on](images/ep1_run1_commentator_drift.png)

*Figure 1. A new screening chat: the agent asks for the PubMed and LibKey connectors to be turned on before it can search.*

---

## 3. Failure log

### Episode 4 — Hidden, non-reproducible search (2 runs)
Identical Message 1 in two fresh chats. We expected the same or a similar search, shown to us for approval.
- **Run 1:** broad all-fields search, no review filter → **149 hits**; batch 1 almost all reviews/editorials/news; **2/40 included**.
- **Run 2:** first ran a 285-hit search and **discarded it silently**, then a narrower [tiab] search with `NOT review[pt]` that dropped "artificial intelligence" → **51 hits**, **10/40 included**.
- Neither search was shown for approval before it ran. Run 1 gave only a paraphrased summary after screening, and its exact string only when we asked; Run 2 showed its query alongside the results, after the search had run. Both offered a better search *after* screening (Run 1: filter reviews; Run 2: add headset terms).
- **Where a person could step in:** a "here is my search string and filters, approve?" checkpoint before any screening. Dropping "artificial intelligence" also partly pre-applied our rule 3 before we gave it, which no reviewer would know without asking.

![Episode 4 Run 1 — PubMed search returned 149 hits](images/ep4_run1_149_hits.png)

*Figure 2. Run 1, batch 1: top 20 of 149 matches, mostly reviews, with one INCLUDE (#9). The search appears only as a paraphrase after screening, followed by an offer to filter out reviews.*

![Episode 4 Run 2 — narrower [tiab] search returned 51 hits](images/ep4_run2_285_to_51_hits.png)

*Figure 3. Run 2, batch 1: a [tiab] query with `NOT review[pt]` and no "artificial intelligence", 51 hits, shown with the results after the search ran. Batch tally before corrections: 8 INCLUDE, 5 UNSURE, 7 EXCLUDE.*

### Episode 5 — Date filter claimed but not applied (2 runs)
Criterion: "published 2015 or later."
- **Same leak in four sessions:** Run 1, PMID 24199656 (2013/2014); Run 2, PMID 24199656 and PMID 23010771 (2012); the resumed session, PMID 25570187 (2014); Part 5 Chat A, PMID 24199656 again (reported unprompted, because we asked for the earliest year).
- In Run 2 the agent said it *set* a 2015 start date, but it *was not applied*. Screening caught these by hand, but the hit counts (149, 51) include an unknown number of out-of-range papers.
- **Where a person could step in:** a check that the reported filters match the returned records, before the counts go into a PRISMA diagram.

![Episode 5 Run 1 — pre-2015 record PMID 24199656 returned despite 2015 date filter](images/ep5_run1_date_filter.png)

*Figure 4. Run 1, batch 2 and follow-up: record 37 (PMID 24199656, 2013/2014) is excluded as pre-2015. Asked for the search, the agent gives the string, "Date from 2015" and 149 results, and flags a "Date filter leak". Running total: 2 INCLUDEs in records 1–40.*

![Episode 5 Run 2 — pre-2015 record PMID 23010771 returned despite 2015 date filter](images/ep5_run2_date_filter.png)

*Figure 5. Run 2, batch 2: record 30 (PMID 23010771, 2012) is excluded as pre-2015. Cumulative: 10 INCLUDE, 3 UNSURE, 27 EXCLUDE. Run 2's earlier leak and its "2015 start date was set" note are in Figure 3.*

### Episode 1 — Commentator drift (2 runs)
- **Run 1 (9 Oct, chat "Along"):** we pasted each Studio instruction in turn, intending to start the task. Every reply was reasonable and the agent offered to start three times, but the session never moved from planning to doing. Nothing on screen showed "task not started."
- **Run 2 (15 Oct):** fresh chat, same three instructions ("Pick a task", "Each team takes one of two paths", "tell me the next steps"). **Same failure:** ranked tasks, an Explore plan and a 10-step checklist, two offers to start, zero work done; it never proposed a research question or a draft search. New: it warned "I'm also the one under evaluation, so watch me critically" — self-aware, but it still didn't act.
- **Telling detail:** its own ideal design in reply 1 was "it proposes search strings and I approve them" — exactly the checkpoint it skipped in Episode 4. It can describe good human control without practising it.
- **Where a person could step in:** after the first offer to start — but the drift is only visible by scrolling the transcript.

### Episode 3 — Resuming across sessions (2 runs, mixed result)
- **Run 1 (15 Oct):** a new chat asked for our submission from prior work. Past-chat search recovered our *plans* but no runs or decisions, because none had happened yet.
- **Run 2 (15 Oct):** "Continue screening where we left off." The agent found the screening chat, reused the 51-hit search, re-applied all three rules, screened the last 11 records, and carried forward the open policy calls (#35, #39). **State recovery worked**, but:
  - it **chose which run to continue** (Run 2, not Run 1) without asking; it told us, but only after deciding;
  - it **re-ran the search** and couldn't confirm the same 11 records were at positions 41–51, so it asked us to check for overlap;
  - it counted policy-dependent records (#43, #50) as **INCLUDE** in the final tally (13), so the headline number depends on decisions we haven't made.
- **Where a person could step in:** a resume prompt ("I found two runs, 149 and 51 hits; which one?") and a saved record list instead of a re-run search.

![Episode 3 Run 2 — resuming screening in a new chat](images/ep3_resume.png)

*Figure 6. Run 2 resume: the agent picks chat "2" (51 hits) over chat "1" (149 hits), screens records 41–51, counts #43 and #50 as INCLUDE pending the #39 policy call (final tally 13 INCLUDE, 4 UNSURE, 34 EXCLUDE), and asks us to check record order. Record 45 (PMID 25570187, 2014) is another pre-2015 leak.*

### Episode 2 — Corrections carry over (worked as expected, 2 runs)
After batch 1 we gave three rules (healthy-only → exclude; non-immersive screen "VR" → exclude; unnamed "AI" → exclude), then asked for the next 20 with no reminder. **Result: 0 rule violations in batch 2 in both runs.** The agent also showed the cost of our rule ("rule 2 removed six records that otherwise met every criterion") and flagged appraisal issues (#20 single-arm called "randomized"). Carryover to the immediate next batch was not a problem; carryover after a gap or a new session is untested.

![Episode 2 Run 1 — three correction rules applied to batch 1](images/ep2_run1_corrections.png)

*Figure 7. Run 1: the rules move #9 (PMID 41245437, screen-based treadmill VR) to EXCLUDE, leaving no INCLUDEs in records 1–20. Batch 2 is in Figure 4.*

![Episode 2 Run 2 — three correction rules applied to batch 1](images/ep2_run2_corrections.png)

*Figure 8. Run 2: the rules change 7 of 20 decisions (now 6 INCLUDE, 2 UNSURE, 12 EXCLUDE). Also visible: the HOLOBALANCE claim made without full text and the #20 "randomized" flag. Batch 2 is in Figure 5.*

---

## 4. Taking it further
- **Repeated runs:** we ran the same task twice in separate sessions; that comparison exposed Episodes 4 and 5.
- **Proposed next step (not carried out):** a 10-minute interview (with consent, anonymised) with someone outside AI/CS who has done a literature review. Which step would you never hand off? What would make you trust a paper you never saw being excluded? How would you want to learn about a mistake? What would you need to defend the result to a supervisor?

## 5. Improvement — a search-approval checkpoint (2 runs, 15 Oct)
**Change (instruction only), added to Message 1:** *"Before searching, show me your exact search string, fields and filters, and wait for my approval. After the search, report the earliest publication year returned."*

| | Before (Runs 1 & 2) | With checkpoint (Chats A & B) |
|---|---|---|
| Search shown before running | No (shown only after running; Run 1's exact string only when asked) | **Yes, both** |
| Hidden search swaps | Yes (285-hit search silently discarded) | **None** — both stopped and asked again when the tool rejected the string |
| Hit counts | 149 vs 51 | **89 vs 83** |
| Overlap in first 20 records | 3 records | **18 of 20** |
| Includes in first 20 | 0 vs 6 | **3 vs 3 (same three PMIDs)** |
| Date leak | Found only by us | Chat A reported it unprompted (2013); Chat B none in top 20 |

![Part 5 Chat A — search shown for approval, rejected by the tool, revised and re-approved](images/part5_run1_approval.png)

*Figure 9. Chat A: the agent proposes a search and waits for approval. The connector rejects the approved string (48 operators, limit 20; wildcards), so the agent proposes a 20-operator revision with broad MeSH headings and asks again. It returns 89 records: 3 INCLUDE (PMIDs 41245437, 41335977, 39810187), earliest year 2013 (PMID 24199656). PMID 41245437, excluded under rule 2 in Runs 1–2, is INCLUDE here.*

![Part 5 Chat B — search shown for approval, rejected by the tool, revised and re-approved](images/part5_run2_approval.png)

*Figure 10. Chat B: the agent proposes a search (noting the tool allows no wildcards) and waits for approval. The approved string is rejected for 36 operators, so the agent proposes a 19-operator revision and asks again. It returns 83 records: 3 INCLUDE, earliest year 2015.*

**Did it help? Yes for control and reproducibility; no for precision.**
- Searches became visible, approvable and nearly reproducible; decisions on the 18 shared records matched.
- Precision didn't improve (3/20): the 20-operator limit forced broad MeSH headings, which pulled in healthy-participant and off-topic studies.
- **New problem:** both approved strings were unrunnable (both over the 20-operator limit; Chat A also used wildcards), so we approved twice — and the second approval asked us to weigh MeSH-vs-free-text recall trade-offs most people can't judge. Approval, but not fully *informed* approval.
- **Corrections don't travel:** PMID 41245437 (screen-based treadmill VR) was EXCLUDE under rule 2 in Runs 1 & 2 but INCLUDE here, because the rules lived only in the earlier chats. A persistent criteria document is the obvious next step.
- **Cost:** one or two extra round-trips per run (~2–5 min).
