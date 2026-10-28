# ADR-005: Hosting — Oracle Cloud Always Free with Home-Server Fallback

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The platform needs one always-on Linux host with: outbound HTTPS only (no
public inbound — SEC-07), ~1 GB RAM headroom (NFR-11), and **$0/month fixed
cost** (NFR-24). A trading bot's failure mode matters: downtime costs
opportunity, not safety (Architecture §7), so best-effort availability with
good alerting is acceptable.

## Decision

**Primary:** Oracle Cloud **Always Free** tier (ARM Ampere A1: up to 4 OCPU /
24 GB RAM, always-free under current terms). **Fallback:** a home Linux
server or repurposed machine, kept deployment-ready (OQ-03).

Operationally: systemd units only, Tailscale-only admin, heartbeat monitoring
(OPS-06) so reclamation/degradation is detected the same day; the DR runbook
(RB-008) restores service to either host within 4 h (NFR-16).

## Consequences

**Positive:** genuinely $0, cloud-grade uptime, no hardware dependence; ARM
capacity is ~10× the platform's needs.

**Negative:** OCI may reclaim *idle* always-free instances (documented
behavior) or change terms; capacity/region availability can be finicky at
signup; account is a single point of failure like any single host.

**Mitigations:** keep sessions visibly active + heartbeat alerts so reclamation
is caught immediately; home-server fallback maintained with quarterly DR
drills (OPS-11); no vendor-specific services used (NFR-15) so the move is a
re-run of RB-008.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Paid VPS (Hetzner, DO, Linode…) | Breaks NFR-24 for zero functional gain |
| GitHub Actions as the host | Scheduled workflows are best-effort, can be delayed/disabled; poor fit for a long-running notifier service; against Actions design intent |
| Home server only | Power/reliability/ISP realities; fine as fallback, not primary |
| Other always-free clouds (AWS/Azure/GCP free trials) | Trials expire; their permanent free tiers are too small for an always-on host |
