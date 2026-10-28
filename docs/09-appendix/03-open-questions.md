# Open Questions

| | |
|---|---|
| **Document ID** | DOC-041 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |

Live register of unresolved decisions. **An open question is a task**: each
has an owner, a needed-by date, and a defined resolution path. Resolved items
move to §2 (with their answer) and spawn ADRs/CT items where applicable.

## 1. Active Questions

| ID | Question | Needed by | Resolve by | Status |
|---|---|---|---|---|
| **OQ-01** | Wealthsimple Trade API: current program status, waitlist, and terms — apply (CT-01) and record actual access timeline | P2 start (2027-01) | Broker dashboard + support; archive terms (CT-02) | ☐ Open |
| **OQ-02** | Fractional-share capability matrix (Wealthsimple live vs. Alpaca paper) — affects position sizing granularity at CAD 2k and adapter capability flags | G1 freeze (2026-12) | Broker docs + sandbox tests; record in adapter capability register | ☐ Open |
| **OQ-03** | Fallback host: which home machine, its network/power reality, and a 1-page provisioning check — makes the [ADR-005](../04-architecture/adr/ADR-005-zero-cost-hosting.md) fallback real rather than theoretical | P2 (2027-01) | Hardware inventory + a timed boot test; feeds RB-008 | ☐ Open |
| **OQ-04** | Company/product name & trademark search for "MapleQuant" (and alternates) before any public artifact | Before repo goes public / any P4 use | CIPO/NUANS search + web search; log result; rename if collision | ☐ Open |
| **OQ-05** | TSX-listed universe data licensing: is there any compliant $0 path (delayed data, redistribution terms)? If no → stays deferred | Only if pursued (P4+) | Data Requirements §7 licensing review; else explicit "deferred" | ☐ Deferred |
| **OQ-06** | MQ-MOM-1 final universe (8–12 ETFs + defensive asset) and starting parameters — must be frozen before the holdout run | G1 (2026-12) | Research on the research window only; freeze commit + rationale note | ☐ Open |
| **OQ-07** | P4 commercialization decision pack: scope options (open-core / content / SaaS / hold), incorporation timing, market sizing, legal budget | 2027-Q4 | Decision pack per [Roadmap P4](../08-planning/01-roadmap.md); counsel scoping; decision record | ☐ Open |
| **OQ-08** | Monitoring backend: Grafana Cloud free tier (limits: series/retention — verify) vs. self-hosted Prometheus+Grafana on the host | P2 (2027-01) | Compare limits vs. metric catalog; one-page note; decide | ☐ Open |
| **OQ-09** | SQLite headroom check: DB size + query times after P2 paper — confirm [ADR-006](../04-architecture/adr/ADR-006-storage-sqlite-first.md) posture or open the Postgres migration path | End of P2 (2027-04) | Capacity review (OPS-21) data; recorded either way | ☐ Open |
| **OQ-10** | Alert-channel redundancy: is single-Telegram an acceptable permanent posture, or add a second channel (e.g., email via free relay)? | P2 (2027-01) | Depends on paper-period alert reliability record; small analysis note | ☐ Open |

## 2. Resolved Questions

| ID | Question | Resolution | Date |
|---|---|---|---|
| — | *(none yet)* | — | — |

## 3. Rules

1. No question may be silently dropped — it is answered or explicitly deferred
   with a re-check date.
2. Resolutions that change **requirements, architecture, or compliance posture**
   spawn: requirements-doc update, ADR if architectural, threat-model check if
   security-relevant (SDLC §5).
3. "Needed by" dates align with the [Roadmap](../08-planning/01-roadmap.md);
   a question blocking a gate becomes the top priority the week before.
