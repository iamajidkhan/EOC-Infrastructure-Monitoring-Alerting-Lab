# SOP — High CPU Incident Response
**Document ID:** EOC-SOP-001
**Version:** 1.0
**Owner:** EOC L1 Analyst
**Last Updated:** April 2026

---

## 1. Purpose

This SOP defines the standard response procedure for a High CPU alert on any monitored host. It ensures consistent, timely triage and escalation by L1 analysts in the EOC.

---

## 2. Scope

Applies to all hosts monitored via Datadog where `system.cpu.user` exceeds defined thresholds.

---

## 3. Alert Thresholds

| Severity | Threshold | Response SLA |
|---|---|---|
| Warning | CPU > 30% avg over 5 min | Acknowledge within 10 min |
| Critical (Alert) | CPU > 40% avg over 5 min | Acknowledge within 5 min |

---

## 4. Response Procedure

### Step 1 — Acknowledge the Alert
- Log into Datadog → Monitors
- Locate the triggered **High CPU - EOC Monitor**
- Note: Host name, current CPU %, alert timestamp
- Acknowledge the alert to stop SLA clock
- Create incident ticket in ServiceNow with:
  - **Category:** Infrastructure
  - **Subcategory:** Performance
  - **Priority:** P2 (High) for Alert / P3 (Medium) for Warning
  - **Description:** Paste alert details from Datadog notification

---

### Step 2 — Initial Triage (L1)

**Check for runaway processes:**

On Linux host:
```bash
top -b -n 1 | head -20
ps aux --sort=-%cpu | head -10
```

On macOS host:
```bash
top -l 1 | head -20
ps aux | sort -rk 3 | head -10
```

On Windows host:
- Open Task Manager → Processes → Sort by CPU
- Note top consuming processes

**Questions to answer during triage:**
- Is this a known scheduled job (backup, antivirus scan, batch process)?
- Is the process a recognised system process or unknown?
- Is CPU sustained high or spiking intermittently?
- Are other hosts affected? (Check dashboard for correlated spikes)

---

### Step 3 — Determine Action

| Scenario | Action |
|---|---|
| Known scheduled job (backup, update) | Document in ticket, monitor until completion, resolve |
| Recognised application consuming high CPU | Notify application owner, monitor, update ticket |
| Unknown or suspicious process | Escalate to L2 immediately — do not terminate |
| Multiple hosts affected simultaneously | Declare P1, escalate to L2/L3, notify team lead |

---

### Step 4 — Escalation Criteria

Escalate to **L2** if any of the following apply:
- CPU remains above threshold for > 15 minutes after acknowledgement
- Unknown or suspicious process identified
- Business-critical application impacted
- Multiple hosts affected simultaneously
- Host becomes unresponsive during investigation

**Escalation message template:**
```
Escalating High CPU incident to L2.
Host: [hostname]
CPU: [value]% for [duration]
Process identified: [process name or "Unknown"]
Initial triage performed: [summary of steps taken]
Ticket: [ServiceNow ticket number]
```

---

### Step 5 — Resolution & Documentation

Once CPU returns to normal:
- Confirm CPU is below warning threshold for at least 10 minutes
- Update ServiceNow ticket with:
  - Root cause (if identified)
  - Actions taken
  - Resolution time
- Resolve the Datadog monitor (click "Resolve" if needed)
- Update shift handover notes

---

## 5. Related Monitors

| Monitor | Datadog Query |
|---|---|
| High CPU | `avg(last_5m):avg:system.cpu.user{*} by {host}` |
| High System Load | Auto-monitor via Datadog host integration |

---

## 6. Contact & Escalation Path

| Level | Role | Contact |
|---|---|---|
| L1 | EOC Analyst (this SOP) | On-shift analyst |
| L2 | Senior Systems Engineer | Via ServiceNow escalation |
| L3 | Infrastructure Lead | Via L2 escalation |

---

## 7. Revision History

| Version | Date | Change |
|---|---|---|
| 1.0 | April 2026 | Initial version |
