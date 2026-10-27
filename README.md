# **FleetWatch**

### FleetWatch is a modern, container-native telematics platform built for the cloud era. It provides real-time medical devices tracking, its health, and devices and assets management with a focus on quick remediation.

### Key Features

- Real-time medical devices tracking
- Device health monitoring
- Device and asset management
- Quick remediation

### Diagram

Planned Architecture:

<img width="768" src="/assets/images/System%20Architecture.png" alt="FleetWatch Architecture" />

### Event flow

1. **Every minute**, EventBridge Scheduler triggers the **simulator Lambda**. It reads each device's state from DynamoDB, applies any injected fault, updates `last_seen` and `status`, and prints one **EMF** (Embedded Metric Format) log line per site, which CloudWatch turns into metrics.
2. Meanwhile the **CloudWatch Agent** on EC2 reports process count, memory and disk usage.
3. **CloudWatch alarms** evaluate those metrics (for example "3 of 5 datapoints breaching").
4. When an alarm changes state, AWS sends an event to the default **EventBridge** bus. A rule matching alarm names that start with `fleetwatch-` invokes the **incident Lambda**.
5. On `ALARM`: the Lambda opens an incident (conditional write, so duplicates are ignored), looks up the runbook, runs it, and logs the result. If there is no runbook, or the runbook fails, it publishes an **SNS** escalation with the severity.
6. When the problem is fixed, the alarm returns to `OK`. The same rule fires again, and the Lambda **closes the incident** and stores `mttr_seconds` and `auto_remediated`.
7. Every hour, the **SLA reporter Lambda** reads closed and open incidents, calculates uptime per site over a rolling 30 days, and publishes metrics.
8. The **dashboard** shows live fleet health, active incidents, MTTR and uptime.

### Possible AWS services used

1. **EC2**
2. **Lambda**
3. **DynamoDB**
4. **CloudWatch**
5. **EventBridge**
6. **SNS**
7. **SSM**

### The Tech Stack used for building the solution

1. **Terraform**
2. **GitHub Actions**
