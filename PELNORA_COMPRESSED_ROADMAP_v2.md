# Pelnora — Compressed Roadmap v2

**Version**: 2 (was v1 from 2026-05-15)
**Updated**: 2026-05-18 (poniedziałek)
**Trigger for v2**: Strategic pivot 2026-05-16 — library-as-primary-feature dropped in favor of knowledge-base-and-agents architecture
**Target launch**: 2026-06-24 (Wednesday — public OSS launch)
**Buffer**: 2 weeks (final QA window 2026-06-25 → 2026-07-08)

---

## Executive summary

Six-week sprint to v0.1 public OSS launch. Pelnora delivers polish-first legal AI assistant with **live access to current Polish law (ELI API)**, **case law cross-referencing (SAOS API)**, **multi-step procedural skills (Procedury)**, **privilege engine with hard gate**, and **audit trail** — running primarily in-browser on user's device.

Strategic positioning: **NOT a template library with AI helper**. Pelnora is a legal agent that writes individualized documents using current statutes and case law, with full provenance. Templates exist as agent reference material, not user-facing primary feature.

Competitive moat (per 2026-05-16 research):
- No other Polish legal tool combines live ELI + SAOS + privilege engine + local LLM (Bielik) + audit
- LEX/Legalis: closed API, no AI integration, no privacy by design
- ChatGPT/Claude generic: training cutoff lag, hallucinated act sigs, no Polish-first
- Existing polish legaltech (DocPlanner, Mecenas, etc.): document management, no agent layer

---

## Sprint cadence (post-Sprint-3-weekend reality)

| Week | Dates             | Sprint | Primary focus                                                        |
|------|-------------------|--------|----------------------------------------------------------------------|
| W1   | 2026-05-14 → 17   | S3     | WebLLM Bielik + privilege Q&A + block segmenter + anonimizator       |
| W2   | 2026-05-19 → 25   | S3 tail| Strategic propagation + docs update + Sprint 4 W3 setup              |
| W3   | 2026-05-26 → 31   | S4 (a) | Knowledge base infrastructure (ELI SDK + SAOS + lookup Procedura)    |
| W4   | 2026-06-02 → 07   | S4 (b) | B2B contract analysis Procedura (proof of concept)                   |
| W5   | 2026-06-09 → 14   | S5     | Cmd+K + autonomy slider + 3 more Procedury                           |
| W6   | 2026-06-16 → 23   | S6     | MCP outbound server + Stripe + Cloudflare Worker + final QA          |
| ―    | 2026-06-24        | ―      | Public OSS launch                                                    |
| W7   | 2026-06-25 → 07-08| Buffer | Hot fixes, community feedback, v0.1.1 planning                       |

---

## Sprint 4 W3 (2026-05-26 → 31) — Knowledge base infrastructure

### Goal
Lay foundation for live polish law access. By end of week, Pelnora has TypeScript SDK for ELI + basic SAOS wrapper + first user-facing Procedura ("ELI lookup tool").

### Deliverables

**Pn 26 — Wt 27 maja wieczór** (~6h CC):
- `src/lib/eli/client.ts` — HTTP wrapper, defensive throttle ~5 req/s, exponential backoff, error handling
- `src/lib/eli/search.ts` — `searchActs({ title, year, inForce, type, ... })` typed query builder
- `src/lib/eli/fetch.ts` — `getActDetails`, `getActText`, `getActFragment(... path notation)`, `getReferences`
- `src/lib/eli/types.ts` — TypeScript types ported from OpenAPI 3.0.3 spec (manual or codegen)
- `src/lib/eli/cache.ts` — IndexedDB layer, 30-day TTL for act texts, indefinite for metadata
- Unit tests for 5 key endpoints

**Śr 28 maja wieczór** (~3h CC):
- `src/lib/saos/client.ts` — SAOS HTTP wrapper, basic search + fetch
- `src/lib/saos/types.ts` — judgment metadata + court taxonomy
- Cache layer (similar pattern to ELI)
- Unit tests

**Cz 29 maja wieczór** (~3h CC):
- `src/lib/knowledge/polishLawTool.ts` — unified tool wrapper for agent
- `polishLawSearch(query, opts)` → returns `KnowledgeResult[]` with ELI URIs + SAOS IDs + relevance scores
- LLMRouter integration: Bielik/Anthropic can `function call` this tool
- Privilege handling: all knowledge fetches are Z-level (public data, no privilege restriction)

**Pt 30 — Sb 31 maja** (~4-5h CC + manual testing):
- UI: new "Procedury" entry in sidebar (sibling to Q&A panel)
- First Procedura: "ELI lookup tool" — Cmd+K or sidebar entry
- Workflow: user types "art. 750 KC" or "ustawa o RODO" → ELI search → display result with text + last-modified-date + ELI URI
- Audit log: every ELI call logged
- Manual testing on 5-10 common queries (KC articles, RODO, KPC, prawa autorskie)

### Acceptance W3
Marta types "art. 750 KC" in Cmd+K → within 2 seconds sees:
- Tytuł: "Umowa o świadczenie usług"
- Tekst: full art. 750 (with par./ust./pkt. structure preserved)
- Ostatnia zmiana: konkretna data + URI ELI
- "Powiązane akty" (referenced/referencing acts) collapsed expandable
- Citation copy: `art. 750 ust. 1 ustawy z dnia 23 kwietnia 1964 r. — Kodeks cywilny (Dz.U. 1964 nr 16 poz. 93 ze zm.)`

---

## Sprint 4 W4 (2026-06-02 → 07) — B2B Contract Analysis Procedura

### Goal
Build first ambitious Procedura — multi-step legal workflow with reasoning + citations. This is the **launch demo material**: shows Pelnora's unique value over ChatGPT/LEX.

### Deliverables

**Pn 2 — Wt 3 czerwca** (~5h CC):
- Procedura UI scaffolding: multi-step workflow with progress indicators
- Audit log per step (which sources fetched, which LLM calls made, which decisions agent took)
- Privilege handling: cały workflow Y-level (local Bielik for parsing + Anthropic with anonimizer for complex reasoning)

**Śr 4 — Cz 5 czerwca** (~6h CC):
- Workflow logic + LLM prompts for "B2B service contract analysis":
  1. User points at .docx (selected from file tree OR paste)
  2. Agent: extract metadata (parties, compensation type, IP clauses, duration)
  3. Agent: ELI lookups (art. 750 KC, art. 22 § 1 KP, art. 50 PrAut, RODO art. 28 if applicable)
  4. Agent: SAOS search for recent SN judgments on reclasyfikacja B2B → umowa o pracę
  5. Agent: identify 3-5 risk points (with severity score) and 2-3 missing clauses
  6. Agent: generate individualized recommendations with citations
- LLM prompt templates for each step (Polish, structured output)

**Pt 6 — Sb 7 czerwca** (~3h CC + manual testing):
- Cloud override flow (when Bielik OOM risk on long contracts → Anthropic with anonimizer)
- Manual testing on 5-10 real B2B contracts (mix of well-written + problematic)
- Tuning prompts based on actual outputs

### Acceptance W4
Marta opens test B2B contract (~10 stron), clicks "Procedury → Analiza umowy o świadczenie usług B2B". Within 30 seconds:
- 3-5 risk points displayed with severity, KC/KP article references, SAOS judgment IDs where applicable
- 2-3 missing clauses identified (e.g. RODO art. 28 powierzenie danych, kara umowna wysokości, ubezpieczenie OC)
- 10+ citations with concrete ELI URIs (no hallucinated sigs)
- Audit log shows all 23+ steps and 0 forced cloud calls (local Bielik handled it)
- Marta can drill into any risk point → see exact KC article + SN judgment text inline

This is the launch demo. Record video for launch post 24 czerwca.

---

## Sprint 5 W5 (2026-06-09 → 14) — Cmd+K + Procedury expansion

### Goal
Add 3 more Procedury + Cmd+K invocation pattern + autonomy slider.

### Deliverables

**Pn 9 — Wt 10 czerwca** (~5h CC):
- Cmd+K menu integration: lists all available Procedury + ELI lookup tool
- Autonomy slider 0-5 (per CLAUDE.md design): 0 = ask before every step, 5 = run end-to-end
- UI consistency with existing sidebar Q&A
- Keyboard shortcuts (Cmd+K for menu, Esc to cancel running Procedura)

**Śr 11 — Cz 12 czerwca** (~6h CC):
- Procedura 2: "Pozew o zapłatę z dowodami" (KPC art. 187)
  - Extract creditor + debtor data from user input
  - ELI: KPC art. 187, art. 50532 (postępowanie nakazowe), art. 339 (czyny niedozwolone if applicable)
  - SAOS: recent rulings on dowody w postępowaniu nakazowym
  - Output: draft pozew with placeholders + evidence checklist + citations

**Pt 13 — Sb 14 czerwca** (~5h CC):
- Procedura 3: "RODO incident response" (powiadomienie art. 33 + reakcja art. 34)
  - User describes incident
  - Agent: assess severity per UODO guidelines + ELI references
  - Output: drafted notification with timeline + risk assessment
- Procedura 4 (if time): "Opinia prawna z citations" — general-purpose legal opinion drafting

### Acceptance W5
Cmd+K shows menu with 5 entries (ELI lookup + 4 Procedury). Marta can invoke any Procedura with autonomy level pre-set. End of week: 4 working Procedury, all with privilege handling + audit.

---

## Sprint 6 W6 (2026-06-16 → 23) — MCP outbound + monetization + QA

### Goal
Final week before launch. Ship monetization layer + outbound MCP + QA polish. No new features after Wt 18 czerwca (code freeze 5 dni before launch).

### Deliverables

**Pn 16 — Wt 17 czerwca** (~6h CC):
- Cloudflare Worker for Anthropic key management (per ARCHITECTURE.md Rule 2)
- BYO-key flow for v0.1 + managed-key flow scaffolding for v0.1.x
- Per-user/per-org rate limiting in Worker
- Worker deployed to production

**Śr 18 czerwca** (~5h CC):
- Stripe integration: signup flow + checkout + webhook → Worker rate limit adjustment
- Pricing v0.1: Free tier (BYO-key, ELI/SAOS unlimited) + Premium tier (managed key, ~99 PLN/miesiąc)
- Code freeze 18 czerwca wieczór

**Cz 19 — Pt 20 czerwca** (~4h CC + Twoje manual QA):
- MCP outbound server: Pelnora exposes Procedury as MCP tools
- Other agents (Claude Code, Cursor, etc.) can connect to Pelnora and call procedures
- Documentation: MCP integration guide

**Sb 21 — Pn 23 czerwca** (~6h Twoje manual QA + bug fixes):
- Full manual QA: every Procedura, every privilege level, every storage adapter
- Bug fix freeze: only critical bugs
- Marketing materials final: launch post (PL + EN), demo video, README polish

### Acceptance W6
Production-ready Pelnora deployed. 4-5 working Procedury. Monetization flow tested end-to-end. MCP outbound documented. Demo video recorded. Launch post ready.

---

## Launch (2026-06-24, środa)

### Distribution

- **OSS repos public**: pelnora-starter (Apache 2.0), pelnora-blocks (Apache 2.0), Pelnora/Bielik-Minitron-7B-v3-Instruct-q4f16_1-MLC (HuggingFace, Apache 2.0)
- **Landing page live**: pelnora.app (or similar)
- **LinkedIn long-form post** (PL): launch announcement with demo video
- **Dev.to / blog (EN)**: technical deep-dive
- **HN submission** (EN, after-noon EST optimal)
- **Polish legaltech community**: reach out to specific people (lawyers' associations, polish AI thought leaders)

### Launch criteria (must-haves)

1. ✓ Bielik works in-browser (~95% success rate first-load)
2. ✓ Privilege engine hard gate functional (R never leaks to cloud)
3. ✓ Anonimizator regex covers PESEL, NIP, REGON, KRS, IBAN, phone, email, postal, ID, passport
4. ✓ Block segmenter handles 80%+ of polish DOCX patterns (regulamins, contracts, formal letters)
5. ✓ ELI live lookup works for 100% of test queries (sample: 30 common KC/KP/KPC articles)
6. ✓ SAOS basic search works (judgment retrieval + display)
7. ✓ At least 3 working Procedury (B2B analysis + ELI lookup + RODO response)
8. ✓ Cloudflare Worker key management deployed
9. ✓ Stripe checkout flow tested with real card
10. ✓ Demo video recorded (3-5 min)
11. ✓ Launch posts drafted (PL + EN)
12. ✓ At least 5 manual QA passes on different DOCX types

### Nice-to-have (not blockers)

- Bielik NER for entity types beyond regex (imiona, firmy, adresy) — deferred to v0.1.1
- More than 4 Procedury — deferred to v0.1.1
- MCP outbound documentation polish — can ship with minimal docs
- More than 1 boilerplate in pelnora-blocks — natural accumulation, not forced

---

## v0.1.1 backlog (post-launch, ~July-August 2026)

### Knowledge layer enhancements
- Bielik NER per-block for entity types beyond regex (imiona, firmy, adresy)
- Hierarchy tree + semantic aggregation in block segmenter
- More Procedury (target 10 total by v0.1.1)
- Komentarze + doktryna integration (negotiate with LEX/Legalis or polish open commentary sources)

### Performance + reliability
- Prompt caching (Anthropic feature, reduces token costs for repeat queries)
- WebGPU streaming for Bielik (better UX on long contracts)
- Offline mode for ELI (precomputed bundle of top 100 acts)

### Library polish
- Schema parser for meta.yaml (currently only structure documented, parser deferred)
- Selector logic in agent: parse `--- WARIANT A ---` markers + apply user choices
- Find&replace placeholders in docx programmatically
- @eigenpal Polish styleId upstream PR (existing bug discovered Sprint 3)

### Anonimizator
- Bielik NER for imiona/firmy/adresy (deferred from v0.1 due to F1 OOM constraint, plan: per-block invocation)
- Multi-match validator policy refinement
- Cloud override audit improvements

### Marketing + ecosystem
- Tauri desktop wrapper (per ARCHITECTURE.md Rule 1+2, infrastructure ready)
- More HF model variants (q4f32_1 for users with more RAM)
- Pelnora Skills marketplace (community-contributed Procedury)

---

## Decision log (reverse chronological — most recent first)

### 2026-05-18 (poniedziałek) — Roadmap v2
- Strategic pivot from 2026-05-16 codified in roadmap structure
- Sprint 4 W3-W4 scope rewrite: knowledge base infrastructure + B2B analysis Procedura
- Sprint 5 W5 expanded: 3-4 Procedury + Cmd+K + autonomy slider
- Sprint 6 W6 unchanged: MCP + Stripe + Worker + QA
- v0.1.1 backlog updated: NER deferred (knowledge base takes priority), more Procedury planned

### 2026-05-16 (sobota) — Strategic pivot: knowledge base over template library
- Trigger: Przemysław's observation "po co wzory, skoro pisma mają być indywidualnie pisane przez agentów"
- Pelnora's competitive moat shifts from "polish legal templates with AI helper" to "AI agent with live access to current polish law + case law + privilege engine + audit"
- pelnora-blocks repo remains as agent reference material, not user-facing primary feature
- Forced curation of 35 boilerplates deprioritized → organic accumulation
- Implementation approach: hybrid (TypeScript SDK native for ELI/SAOS core + sejm-mcp as external MCP for less-common queries)
- First Procedura scope: ELI lookup tool in W3 (foundation), B2B contract analysis in W4 (proof of concept)

### 2026-05-16 (sobota) — Polish legal nomenclature precision (library convention)
- Rule: never use generic terms; always cite KC/KP/KPC article
- Specific corrections: "umowa o świadczenie usług" (art. 750 KC, B2B) ≠ "umowa zlecenia" (art. 734 KC, civil with ZUS)
- Documented in pelnora-blocks/README.md naming convention section

### 2026-05-15 (piątek) — Architectural Findings F1/F2/F3
- F1: 16 GB Mac OOM ceiling = mandates block-aware paradigm for all LLM operations
- F2: WebLLM json_object schema rejection = prompt-only structured output + defensive parsing
- F3: Anonimizator regex-only at v0.1 = NER deferred to v0.1.1, messaging adjusted in cloud override dialog
- Documented in PELNORA_ARCH_FINDINGS.md

### 2026-05-15 (piątek) — Bielik NER reverted, F1/F2 discovered
- Attempted: Bielik NER full-document → WebGPU device-lost crash
- Reverted commit, identified F1 (memory) and F2 (json_object) as root causes
- NER deferred to v0.1.1 with per-block paradigm

### 2026-05-08 — Bielik MLC integration path B viable
- Self-conversion via Claude Code 2-3 days viable
- Default model 4.5B v3 q4f16_1 (~3GB), opt-in 11B v3 q4f32_1

### 2026-05-10 — Switch to Bielik-Minitron-7B
- New base: Mistral 7B v0.2 (pruned by NVIDIA Minitron from 11B base)
- HF publish: Pelnora/Bielik-Minitron-7B-v3-Instruct-q4f16_1-MLC
- 8/8 smoke tests pass, art. 101² KP test critical for B2B use case

### 2026-05-10 — Gemma 4 DEFER verdict
- Cache 4.85 GB vs Bielik 2.5 GB
- Runtime crashes (onnxruntime#26732, transformers.js#860)
- Polish legal knowledge gap (test #6 fail)
- Custom tool-call parser needed
- Re-eval narrow scope: OCR-only attachment v0.1.1+

---

## Risk register

### High impact / non-blocking for launch

**R1 — ELI API rate limits or downtime (NEW for v2)**
- Risk: official docs don't specify rate limits; could be subject to change without notice
- Mitigation: defensive throttle ~5 req/s in SDK + aggressive caching (30-day TTL) + offline mode planned v0.1.1
- Plan B: precomputed bundle of top 100 acts shipped with app for offline fallback

**R2 — SAOS data quality / coverage (NEW for v2)**
- Risk: case law database has 178K+ judgments but coverage uneven (especially older + lower courts)
- Mitigation: focus initial Procedury on SN + appeal court judgments where coverage is best
- Plan B: messaging in UI about coverage limitations ("Wyniki dotyczą wyroków SN i sądów apelacyjnych. Niższe instancje mogą być niedostępne.")

**R3 — Bielik OOM on long contracts (KNOWN from Sprint 3)**
- Risk: documents >10 pages with full B2B analysis may exceed 16 GB memory budget
- Mitigation: block-aware paradigm everywhere; per-block LLM calls; cloud override dialog when local fails
- Plan B: gradient degradation — if local fails, offer "Wyślij do chmury z anonimizatorem" with explicit user consent

### Medium impact

**R4 — Strategic pivot codification lag**
- Risk: docs not fully updated to reflect strategic pivot; CC works with stale context
- Mitigation: roadmap v2 (this file) + architecture map review + briefing file all updated this week
- Plan B: in CC executive prompts, explicit reference to "strategic pivot 2026-05-16" with summary inline

**R5 — Launch slippage**
- Risk: 6-week timeline tight; one bad week could push launch
- Mitigation: 2-week buffer (2026-06-25 → 07-08); code freeze 5 days before launch (Wt 18 czerwca)
- Plan B: launch with 3 Procedury instead of 4 if W5 slips

### Low impact / managed

**R6 — Bielik NER absence in v0.1**
- Risk: cloud override dialog has incomplete anonymization messaging (regex only, no NER)
- Mitigation: explicit messaging in dialog about what's protected vs what's not
- Plan B: encourage Marta to manually redact in editor before cloud send (workflow already supports this)

**R7 — MCP outbound complexity**
- Risk: MCP server requires careful design to expose procedury as tools without leaking privilege bypass
- Mitigation: launch with MCP outbound feature flagged as "experimental"; full polish in v0.1.1
- Plan B: skip MCP outbound for launch, ship in v0.1.1

---

## Open questions (resolve in W2-W3 week)

1. **Cmd+K UI**: invoke Procedury vs. lookup tool in same menu, or separate? Lean toward unified.
2. **Procedura output format**: drafted document inline vs. side panel vs. modal? Lean toward side panel (preserves editor focus).
3. **Audit log UI**: how much detail visible to user? Default collapsed, expandable? Per-Procedura toggle?
4. **Free vs Premium pricing**: 99 PLN/mo Premium is placeholder. Validate with 3-5 lawyers before launch.
5. **MCP outbound scope**: which Procedury exposed externally? All 4? Or curated subset?

---

## Authors / contributors

Built by Przemysław Kołakowski (radca prawny + indie hacker) with Claude (Anthropic) as AI assistant. SpeakLeash team for Bielik base model. Apache 2.0 license throughout.

Roadmap updates: 2026-05-15 v1 → 2026-05-18 v2.
Next update: post-launch (2026-06-25) reviewing what shipped vs planned.
