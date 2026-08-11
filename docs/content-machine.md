> **Superseded 2026-08-10:** the canonical copy of this doc now lives in the vault (`knowledge/harry-hunt-jr/hhj-series-story.md` / `knowledge/content-factory/content-machine-design.md`, with reconciliation). This repo copy is the frozen remote-session original.

# The A2G Content Machine

**Scope:** the full production system for story-driven and repurposed content
across all A2G brands — HHJ (the transformation series), H2 The Scientist, TMV,
and client brands (Aniba / Aniba & The Sol Starz, Leo Sun Processing). Designed
whole, up front, per Harry's directive: *"do not build it as we go."*

Written 2026-08-10 from a remote session. Sources: the May 11 Lane 1 decision
log entry, HHJ Content Pipeline v0.1 project spec, Engineering Operating Rules
§2 references, Cold-Start Build vault doc (Aug 6), the H2/HHJ brand pages, and
tonight's story-development session (see `ai-movie-journal.md`).

---

## 0. System health check — findings before recommendations

Harry asked for a health check before architecture. Honest accounting:

### What I could verify (Notion)

| Finding | Status |
|---|---|
| Lane 1 architecture (Studio renders, Mini deploys via GHL API) decided May 11, project marked **Live** | ✅ Sound, and this plan builds on it rather than replacing it |
| Engineering Operating Rules §2 (Studio = on-demand appliance, Mini = always-on host) | ✅ Still the right split; nothing here violates it |
| **Two** "Content Pipeline" databases exist in Notion (created Mar 24 and Apr 5) | ⚠️ Probable duplicate — one should be marked canonical, the other archived. Classic instance of the stale-artifact problem Harry named. |
| Content Console (Apr 21) says "YouTube evaluation gate: **Aug 2026**" | ⚠️ That gate is *now*. This plan effectively answers it (see §9) — the Console page should be updated, not left as a stale instruction. |
| HHJ Pipeline v0.1 page and Cold-Start Build page are **archived** in Notion | ⚠️ Consistent with the July vault migration — but it means Notion no longer shows current pipeline state. See canonicality note below. |
| Show Formats project: Status **Paused**, last review March | ⚠️ Its H2 show definitions are still load-bearing for this plan; the project status should say "reference doc," not sit as an overdue-review item. |
| No page found for "Vault Structure Scalability Review" | ⚠️ The session that most recently reshaped the system left no record visible from Notion. Its outcome presumably lives in the local vault only. |

### What I could NOT verify (and must not pretend to)

This remote container has **no `~/OS` vault** — only the a2g-demos repo and
Notion. I cannot see current NOW files, handoffs, MOCs, or the scalability
review's actual output. The April convention said *"Notion is canonical, the
vault is the working copy,"* but the July migration moved working records
vault-ward — so **canonicality is now ambiguous when viewed from Notion, and
that ambiguity is precisely the failure mode Harry described** (parallel
truths, old files never cleaned up).

### Prescribed: the freshness protocol (run from a local session, ~30 min)

1. **One NOW per domain.** Any second status-bearing file in a domain is either
   merged or archived the day it appears.
2. **Supersede, don't duplicate.** New versions replace in place; the old
   version moves to the frozen archive with a pointer forward. No `-v2`, no
   `-final`, no parallel copies.
3. **A canonicality manifest** — one short file at the vault root naming, per
   domain: the system of record, the working copy, and the sync direction.
   The content machine below assumes this exists; it is the cure for
   "updated too many things in the same domain."
4. **Weekly drift check** — the Delta 1 "Vault Structure Enforcer" concept from
   April, resurrected as a night-shift job (§7): report-only, proposes
   cleanups, never auto-deletes.
5. **Immediate cleanups from this audit:** merge/archive the duplicate Content
   Pipeline DB; update the Content Console's YouTube gate line; restate the
   Show Formats project as reference material; write a Notion stub pointing to
   wherever the Vault Structure Scalability Review outcome actually lives.

---

## 1. Where things live — the system-of-record map

One rule: **every artifact type has exactly one home, and everything else
points at it.**

| Artifact | System of record | Why |
|---|---|---|
| Brand identity, decisions, project status, logs | **Notion** | Already the brain; MCP-reachable from every Claude surface |
| Story canon (series bible, character sheets, arc/thread state, episode ledger) | **Git repo** (`a2g-content-machine`, new, private) | Versioned, diffable, readable by local *and* remote Claude sessions — Notion can't hold pipeline-consumed JSON well, and the vault isn't reachable remotely |
| Pipeline code (scripts, prompts, style contracts, schemas) | Same git repo | Code belongs in git; prompts and style contracts *are* code |
| Working drafts, scratch, journals | **Local vault** (`~/OS`) | Existing convention; private by default |
| Media files (renders, audio, finals) | **Studio/Mini shared folder** under the existing `/Brands/<BRAND>/` convention with `_inbox/_work/_outbox` per brand | Already the Lane 1 design; media never goes in git or Notion |
| Schedule + publish state | **GHL Social Planner** (per-brand sub-accounts) | Already the deploy fabric; approval queue lives here |

The repo is the piece that makes remote sessions (like this one) full
participants: story canon + prompts in git means any Claude surface can write
an episode draft, and only the Studio can render it. **a2g-demos stays what its
README says it is — client demos.** The machine gets its own repo.

---

## 2. The lanes — one machine, four content shapes

The May decision already named lanes. This plan completes the set:

| Lane | Shape | Brands | Status |
|---|---|---|---|
| **Lane 1** | Long-form talking → transcript-clipped multi-format | HHJ (pilot), later BTG, Aniba | Designed May, v0.1 Live |
| **Lane 1B** | CSV-bank scheduled awareness posts | Leo Sun Processing | Designed, cheap to stand up after Lane 1 Phase 1 |
| **Lane 2** | Music-demonstration content (human-picked cut points) | TMV, H2 performance | Deliberately deferred |
| **Lane 3** *(new)* | **Story-driven generative** — journal → episode → comic panels → light animation → multi-platform | HHJ series (H2 cameos) | This document |
| **Lane 4** *(new, thin)* | Written-first — Substack/LinkedIn essays derived from Lane 3 chapters and Lane 1 transcripts | HHJ | Mostly prompt work, no new infra |

**The critical shared insight: all lanes converge at the same last mile** —
manifest JSON in a brand `_outbox` → Mini POSTs to GHL → approval queue → out.
Build the last mile once (it's HHJ Pipeline v0.1 Phase 3, already specced) and
every lane, every brand, every client inherits it. That is also the direct
answer to "can Claude push through GHL": **yes — Claude-written scripts on the
Mini already own that POST in the existing design.** Remote sessions produce
*payloads*; only the Mini holds credentials and fires deploys. Keep it that
way — one deploy point, one audit log, one place to revoke.

---

## 3. Lane 3 — the story pipeline, stage by stage

With the automation / human-in-the-loop boundary marked for every stage.
Legend: 🤖 fully automated · 🌙 automated overnight, reviewed in the morning ·
👤 human decision, machine-assisted.

| # | Stage | What happens | Who |
|---|---|---|---|
| 1 | **Capture** | 3-min voice memo; fixed closing prompt. Drops into vault + syncs to shared folder | 👤 (the only daily human duty) |
| 2 | **Extract** | Whisper on Studio → transcript → structured JSON (events, emotional shape, people, threads touched) | 🌙 |
| 3 | **Adapt** | Read story bible → map the day's emotional truth onto the next beat of the active arc → episode outline + panel descriptions | 🌙 drafts, 👤 approves (this is a *creative* decision — the machine proposes, Harry disposes) |
| 4 | **Continuity** | Resolve characters/locations/props against the bible; flag new entities for naming | 🤖 with 👤 sign-off on new entities |
| 5 | **Render** | Panel prompts + identity kit + style contract → images (multiple takes per panel) | 🌙 |
| 6 | **Assemble** | Layout, lettering, caption boxes; light animation pass (parallax, camera moves) on the strongest 1–2 panels; score from the H2 track library; narrator VO recorded by Harry in batch | 🌙 assembly, 👤 VO |
| 7 | **Review gate** | Morning queue: approve / reshoot / discard / keep-private per episode | 👤 — never bypassed |
| 8 | **Package** | Per-platform crops + captions + manifest JSON → `/Brands/HHJ/_outbox/` | 🤖 |
| 9 | **Deploy** | Mini → GHL API → platform approval queue → scheduled | 🤖 (GHL approval step remains a second 👤 checkpoint on mobile) |
| 10 | **Learn** | Log takes, costs, failures, approve/discard decisions → technique journal (feeds the HHJ YouTube teardowns) + preference dataset | 🤖 |

Two human moments per day — the evening memo and the morning review — plus a
weekly batch VO session. Everything else is machine time, most of it at night.

**Storyboard-first, per Harry:** before any of stages 2–10 are code, the next
concrete step is storyboarding 3 episodes by hand (see build order, §10). The
pipeline automates a process that must first exist manually.

---

## 4. The story bible — schema sketch

Lives in the new repo as versioned JSON/Markdown. The pipeline's memory.

```
series.md        premise, thesis, tone rules, what-fun-means, hard walls
                 (no money numbers, no house-as-home; basement-as-lab is IN)
characters/      hhj.md (face; the builder), h2.md (goggles; existing kit specs),
                 supporting cast (consent-gated)
world/           the-lab.md (gutted basement → rebuilt across the season —
                 the set literally heals as the real rebuild progresses),
                 machines/ (one file per system Harry ships; each new real
                 system = a new machine drawn into the lab)
arcs/            arc-01.md … with beat ledgers: planned / drafted / published
episodes/        e001.json … (source memo ref, beats, panels, takes, status)
style/           style-contract.md, palettes, lettering, per-platform specs
threads.json     open narrative threads ← what stage 3 reads every night
```

The `machines/` convention operationalizes the "AI as lab equipment, not
protagonist" rule: when Harry ships a real system (research formatter, booking
flow, night-shift runner), it enters the story as a drawn machine — content and
world-building from work that was happening anyway.

---

## 5. Multi-brand isolation — the client-safety wall

Aniba and Leo Sun content will run through the same machine while HHJ story
content is in flight. The isolation rules, from day one:

1. **Filesystem:** everything lives under `/Brands/<BRAND>/` — no shared scratch
   across brands, ever. A render job reads from exactly one brand tree.
2. **Credentials:** one GHL sub-account per brand (already the design); the
   Mini's deploy script maps `_outbox` folder → `locationId` from a single
   routing table. No credential ever appears in a prompt or a repo.
3. **Voice/style:** per-brand style contracts + GHL Brand Voice per sub-account.
   The same adapt-stage prompt never serves two brands; prompts are files in
   per-brand directories.
4. **Review:** client-brand content gets its own morning queue, and GHL's
   External Approval Flow (already flagged in v0.1 Phase 1) gives clients
   final sign-off on their own content — A2G approves internally, client
   approves externally.
5. **The story stays home:** Lane 3 generative treatment is *not* offered to
   client brands until it has shipped publicly on HHJ for a full arc. HHJ is
   the pilot brand precisely so mistakes land on the house brand.

---

## 6. Local models vs. hosted — what the Studio actually does

The M1 Max / 64GB Studio is the arbitrage (the May decision said it: idle
capacity vs. creators paying SaaS-per-minute). Realistic split:

**Local (Studio), tonight-capable:**
- Whisper Large-v3 — transcription (already in the v0.1 spec)
- FFmpeg — all assembly, crops, caption burn-in, light animation passes
- Image generation drafts — Flux/SDXL-class models run acceptably on 64GB for
  *exploration takes*; cheap variety, zero marginal cost
- Embeddings + retrieval over the story bible and technique journal
- Bulk mechanical LLM passes (transcript cleanup, tagging, caption drafts)
  with a local Qwen/Llama-class model

**Hosted frontier, where quality is the product:**
- Stage 3 Adapt — the storytelling. This is the make-or-break stage; do not
  economize here.
- Final panel renders where character consistency matters (hosted models with
  reference conditioning currently beat local for identity lock)
- The eventual image→video passes

**Rule of thumb: local for volume and drafts, hosted for judgment and finals.**
Every hosted call goes through the per-night budget cap, in code, before the
first unattended run.

---

## 7. The night shift — what runs while Harry sleeps

One runner (launchd on Studio per Engineering Operating Rules §2 — on-demand,
fired nightly, not always-on), a job registry, per-job budget caps, one morning
digest. Jobs in priority order:

1. **Lane 3 nightly:** stages 2–6 for yesterday's memo → morning review queue
2. **Render retries** from yesterday's "reshoot" verdicts
3. **Research feed:** the existing research formatter run, summarized into the
   morning digest (this is already one of Harry's built systems — first tenant
   to migrate onto the shared runner)
4. **Drift check** (weekly): the Delta-1-style vault/repo shape report
5. **Idea bench** (budget-capped, lowest priority): overnight side-mission
   drafts — "what would happen if…" one-pagers for the interactive/challenge
   content, never auto-published

Failure discipline: every job degrades to a smaller deliverable rather than
silence (animation fails → stills; render fails → text-card episode; anything
fails → it says so in the digest). Silent success-only logging is how
unattended pipelines rot.

---

## 8. Red team register — what gets adversarially checked, and when

| Surface | Check | When |
|---|---|---|
| Brand positioning | Does any H2-adjacent output imply AI touched the music? (the Enemy line) | Every episode, review gate |
| Likeness/consent | Any drawn character mappable to a real person without recorded consent? Wife/family appearing beyond agreed walls? | Every episode |
| Client cross-contamination | Any brand's voice/asset/credential appearing in another brand's tree or output | Weekly drift check + deploy-script assertion |
| Story walls | Money numbers, house-as-home content leaking in via the journal | Adapt-stage system prompt + review gate |
| Platform compliance | AI-content disclosure flags per platform; GHL rate limits (IG 25/day, TikTok 15/day — fine at this scale, matters when client brands stack) | Package stage, automated |
| Cost | Per-night cap enforced in code; digest reports spend vs. cap | Nightly |
| Prompt-injection | Journal transcripts and audience-submitted challenge ideas are untrusted input to the adapt stage — the night runner must treat them as content, never as instructions | Design-time + adapt-stage guard |
| The archive | A detailed narrated record of Harry's life accumulates in the repo/vault — retention, encryption, and who-can-read decisions made *before* it's large | Once, this month |

---

## 9. How the media types fit — one spine, many cuts

The unit of production is the **chapter** (roughly weekly), assembled from the
nightly episodes. Everything else is a derivative cut — nothing is written
twice:

```
nightly episodes (private, Lane 3)
        │ best material, weekly
        ▼
   THE CHAPTER
        ├─ Short-form (IG/TikTok/YT Shorts): the dramatized comic episode — the SHOW
        ├─ HHJ YouTube long-form: the TEARDOWN — Harry's face, "here's what I
        │    built this week and how," Lane 1 clips it into derivative shorts
        ├─ Substack (Lane 4): the written chapter — story panels inline,
        │    technique notes below the fold, one essay serving both audiences
        └─ H2 channels: any music made along the way, plus the H2 cameo cuts —
             the character driving listeners to the score
```

The YouTube long-form *is* the Aug 2026 evaluation-gate answer: HHJ YouTube
launches as the teardown channel for the series, which gives it a format, a
cadence source, and a Lane 1 clip supply in one move — and it's exactly the
"show the system, polished enough for tech people" layer Harry described.
The show attracts; the teardown converts; A2G Systems closes.

Interactive layer (Harry's "fun," phase 2 of publishing): audience-submitted
side missions — "what should H2 try to build next?" — voted challenges,
fail-counters on screen. These feed the idea bench (§7) and are treated as
untrusted input (§8).

---

## 10. Build order — nothing built as-we-go

**Step 0 — Health pass (local session, 30 min).** The freshness protocol +
cleanups from §0. The machine does not get built on an unaudited substrate.

**Step 1 — Story before software (this week, no code).**
Series bible v1 (premise, walls, tone, the three-character question parked as
an open thread) + hand-storyboard **three episodes** in a chat UI. Decides
title, visual grammar, and whether the storm is the cold open — by making
pages, not by deliberating.

**Step 2 — Repo + last mile (one weekend).**
Create `a2g-content-machine`; commit bible, style contracts, prompts, schemas.
Stand up the Lane 1 v0.1 prerequisites that are still open (shared folder,
GHL Phase 1 for HHJ, Mini deploy script + logging). This serves every lane.

**Step 3 — Lane 3 skeleton (next 2 weekends).**
Memo → transcript → adapt draft → rendered panels → morning review folder.
Stills only. Runs nightly for 7 consecutive nights before anything else is
added. H2 asset kit built in parallel (turnarounds, expression sheet, both
room backplates — Blue Room first, in its gutted state).

**Step 4 — Bank, then launch.**
Produce nightly until ~6 chapters are banked. Launch the short-form series and
the first YouTube teardown in the same week. Counter starts.

**Step 5 — Widen.**
Lane 1B (Leo Sun), Aniba via Lane 1 copy-paste, light animation upgrades,
interactive challenges, and the night runner absorbing more tenants.

**Step 6 — Productize** (unchanged from `ai-movie-journal.md` §11 Phase 5).

---

## 11. Open questions routed to Harry

1. The **three-character relay** (H2 builds → student-scientist Harry → HHJ the
   executive presenting, Jobs-style) — parked in the bible as an open thread;
   the three-episode storyboard in Step 1 is where it gets tested, not debated.
2. **Series title** — "Confessions of a Recovering Musician" is thesis, not
   title (Harry: it's *transformational*, not recovering). Candidates should
   come out of Step 1 storyboarding.
3. **GHL sub-account inventory** — which brands already have sub-accounts +
   connected socials? Determines Phase 1 effort. (Answerable only from a local
   session / GHL login.)
4. **Where the Vault Structure Scalability Review actually landed** — its rules
   should be reflected in the canonicality manifest before Step 2 creates yet
   another repo. If the review changed what's canonical, this doc yields to it.