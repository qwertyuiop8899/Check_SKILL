---
name: check
description: Perform a strict read-only audit and diagnostic analysis of systems, VPS metrics, codebases, or logs without modifying anything or proposing action plans. Trigger when the user uses /check, check, /check deep, or check deep, or asks for a read-only inspection of CPU, memory, disk space, network traffic, service failures, or logs.
---
# Check (Read-Only Audit & Diagnostics)

Execute a strict read-only inspection of the target system, VPS, service, logs, or codebase, delivering only the diagnostic results at the requested depth without making changes or proposing remediation plans.

## When to Use

- The user invokes `/check` or `check` for a standard, high-level diagnostic overview.
- The user invokes `/check deep` or `check deep` for an exhaustive, granular technical inspection.
- The user requests an audit of VPS/server health (CPU, RAM, disk space, network traffic), service failures, or error logs without wanting modifications or action plans.

## Core Constraints (Strict Read-Only Mode)

1. **Zero Modifications:** Do not create, edit, overwrite, or delete any files, configurations, or resources.
2. **Non-Destructive Commands Only:** When generating or running diagnostic commands (if operating in an agentic IDE/terminal environment), restrict exclusively to read-only inspection tools (`top`, `htop`, `df`, `free`, `vmstat`, `iostat`, `ss`, `netstat`, `vnstat`, `systemctl status`, `journalctl`, `docker ps`, `docker stats`, `docker logs`, `pm2 status`, `tail`, `grep`).
3. **Results Only - No Action Plans:** Output only the observed state, metrics, and root-cause findings. Do not include remediation roadmaps, step-by-step fix plans, refactoring proposals, or "Next Steps" sections unless the user explicitly asks for a fix in a separate prompt.

## Depth Modes

### 1. Standard Mode (`/check` or `check`)

Provide a concise, scannable executive summary of current health and anomalies:

- **Overall Status:** Single-line indicator (`HEALTHY`, `WARNING`, or `CRITICAL`).
- **Key Metrics Snapshot:** Compact summary of CPU load, memory/swap usage, disk capacity, and network traffic anomalies.
- **Active Failures:** Bullet list of crashed services, failed units, restarts, or recent critical log errors.

### 2. Deep Mode (`/check deep` or `check deep`)

Provide a comprehensive, low-level diagnostic breakdown:

- **Granular Resource Metrics:** Detailed breakdown of CPU load averages (1m/5m/15m), top consuming processes, memory/swap allocation, inode and filesystem usage, I/O wait, and network interface throughput/connections.
- **Service & Container State:** Full status of active daemons, Docker containers, or process managers (e.g., PM2, systemd), highlighting exit codes, restart counts, and resource limits.
- **Log & Failure Forensics:** Extract exact error timestamps, stack traces, recurring warning patterns, and root-cause correlations across system and application logs.
- **Code/Config Anomalies (if inspecting a repo):** Pinpoint exact files, line numbers, and technical bottlenecks causing issues.

## Diagnostic Workflow

### Step 1: Identify Context & Environment

Determine how the target is being inspected:

- **Provided Data (Logs/Metrics/Code in prompt):** Analyze the pasted output, log files, or codebase directly.
- **Local/Connected Workspace (Agentic IDE/CLI):** Run read-only diagnostic commands to gather live telemetry.
- **Remote VPS (No direct shell access):** Provide a single, copy-pasteable read-only diagnostic command for the user to run on the VPS and analyze the returned results immediately.

#### Read-Only VPS Diagnostic One-Liner (Reference)

    echo "=== SYSTEM & CPU ===" && uptime && lscpu | grep "Model name\|CPU(s):" && \
    echo "=== MEMORY ===" && free -h && \
    echo "=== DISK ===" && df -hT -x tmpfs -x devtmpfs && \
    echo "=== TOP PROCESSES ===" && ps aux --sort=-%cpu | head -n 6 && ps aux --sort=-%mem | head -n 6 && \
    echo "=== NETWORK & PORTS ===" && ss -tulnp && ip -s link && \
    echo "=== FAILED SERVICES ===" && systemctl --failed --no-pager && \
    echo "=== DOCKER (IF PRESENT) ===" && (docker ps -a && docker stats --no-stream 2>/dev/null || true) && \
    echo "=== RECENT CRITICAL LOGS ===" && journalctl -p 3 -xb -n 30 --no-pager 2>/dev/null

### Step 2: Analyze Pillars

Evaluate the target across four diagnostic pillars:

1. **Compute & Memory:** CPU load vs. core count, steal time, RAM saturation, OOM killer invocations, and swap thrashing.
2. **Storage:** Disk usage percentages (flagging partitions above 80%), inode exhaustion, and disk I/O bottlenecks.
3. **Network & Traffic:** Open listening ports, unusual connection states (`TIME_WAIT`, `SYN_RECV`), bandwidth spikes, and packet drops/errors.
4. **Failures & Logs:** Systemd failed units, container crash loops (`OOMKilled`, non-zero exit codes), authentication failures, and application stack traces.

### Step 3: Format the Report

Deliver the findings strictly using the structure below. Stop immediately after the findings.

    ## [STATUS: HEALTHY | WARNING | CRITICAL] - Target Summary

    ### Metrics Overview
    - **CPU:** <load / usage details>
    - **Memory & Swap:** <RAM / swap details>
    - **Disk:** <storage / inode details>
    - **Network:** <traffic / socket details>

    ### Failures & Log Findings
    - <Timestamp / Service / File>: <Exact finding and technical root cause>

## Gotchas

- **Never drift into problem-solving:** Stop strictly after reporting the diagnostic facts. Do not add action items or remediation suggestions.
- **Never alter state to test a hypothesis:** Do not restart services, clear caches, rotate logs, or modify files.
