# MS SQL Server: HA/DR, Always On, Replication, Log Shipping, Security & Maintenance
## Senior Database Administrator Study Guide

---

## Table of Contents

- [Introduction](#introduction)
- [Foundational Concepts](#foundational-concepts)
- [HA/DR Fundamentals](#hadr-fundamentals)
- [Always On Availability Groups](#always-on-availability-groups)
- [Replication](#replication)
- [Log Shipping](#log-shipping)
- [Security Basics](#security-basics)
- [Advanced Security](#advanced-security)
- [Maintenance](#maintenance)
- [Monitoring & Troubleshooting](#monitoring--troubleshooting)
- [Hands-on Scenarios](#hands-on-scenarios)
- [Interview Questions](#interview-questions)

---

## Introduction

### Overview of Topic

This study guide addresses the critical pillars of enterprise SQL Server administration: **High Availability (HA), Disaster Recovery (DR), business continuity, security hardening, and proactive maintenance**. These capabilities form the backbone of mission-critical database infrastructure supporting 24/7 operations across organizations of all sizes.

The topics covered represent a progressive journey from foundational concepts through advanced production strategies:

1. **HA/DR Fundamentals** – The strategic framework and measurable objectives (RPO/RTO/SLA)
2. **Always On Availability Groups** – Modern, integrated Microsoft approach to HA/DR
3. **Replication** – Asynchronous data distribution for scalability and resilience
4. **Log Shipping** – Lightweight, cost-effective DR mechanism
5. **Security Basics & Advanced Security** – Identity management through cryptographic controls
6. **Maintenance & Monitoring** – Proactive health and performance assurance

### Why It Matters in Modern Database Platforms

In 2024+, database unavailability is not merely a technical problem—it directly impacts:

- **Revenue**: Every hour of downtime costs organizations tens of thousands to millions of dollars
- **SLA Compliance**: Service level agreements typically require 99.9-99.99% uptime (43 minutes to 4 seconds of acceptable downtime per month)
- **Data Integrity**: Security breaches and audit compliance violations carry regulatory penalties
- **Competitive Position**: Customers expect always-on, responsive applications
- **Regulatory Mandates**: GDPR, HIPAA, SOX, and industry-specific regulations impose strict data handling requirements

SQL Server's HA/DR ecosystem has matured significantly:
- Availability Groups provide *synchronous replication with automatic failover*
- Distributed AG extend HA across geographic regions
- Encryption technologies (TDE, Always Encrypted) are now expected, not optional
- DMVs and Extended Events provide deep observability without performance penalties

Senior DBAs must master these technologies to architect resilient, compliant systems from day one.

### Real-World Production Use Cases

1. **Financial Services (24/7 Trading Platforms)**
   - Requires sub-second RPO and RTO measured in seconds
   - Uses Availability Groups with synchronous commit across multiple data centers
   - Implements Always Encrypted for sensitive customer data and compliance
   - Maintains multiple geographic failover sites with near-zero data loss tolerance

2. **Healthcare (Electronic Medical Records – EMR)**
   - HIPAA compliance mandatory; audit trails are critical
   - DR site required; typical RPO < 1hour, RTO < 4 hours
   - Log Shipping or Replication for warm standby scenarios
   - Row-level security masks PHI from unauthorized users

3. **Retail/E-Commerce**
   - High-traffic read scenarios (product catalogs, inventory)
   - Read-only replicas improve query performance during peak sales
   - Distributed AG spans US and EU data centers for low latency
   - Snapshot replication for data warehouse and analytics

4. **Enterprise SaaS**
   - Multi-tenant architecture with strict data isolation
   - Availability Groups for automatic failover with zero data loss
   - Cross-region DR for disaster scenarios
   - TDE + Always Encrypted protect customer data at rest and in transit

5. **Government/Defense Contractors**
   - Stringent audit requirements (every operation logged and auditable)
   - Geographically separated DR sites (sometimes mandated)
   - Extended Events for forensic analysis
   - Row-level security enforces data compartmentalization

### Where It Typically Appears in Enterprise Architecture

```
┌─────────────────────────────────────────────────────┐
│         Application Tier (Web/Services)             │
└──────────────────┬──────────────────────────────────┘
                   │
        ┌──────────▼────────────┐
        │  Connection Pooling   │
        │  & Routing (Always On │
        │  Listener / AG DNS)   │
        └──────────┬────────────┘
                   │
        ┌──────────▼────────────────────────────────┐
        │     SQL Server Instance (Primary)         │
        │  ┌──────────────────────────────────────┐ │
        │  │  Database (HA/DR Protected)          │ │
        │  │  ├─ Availability Group (Sync Commit) │ │
        │  │  ├─ TDE Enabled (Encryption)         │ │
        │  │  ├─ Audit Policy (SOX/HIPAA)        │ │
        │  │  └─ RLS (Row-Level Security)         │ │
        │  └──────────────────────────────────────┘ │
        │  ┌──────────────────────────────────────┐ │
        │  │  Maintenance Jobs (Index, Stats)     │ │
        │  │  Extended Events (Tracing)           │ │
        │  │  SQL Server Agent (Monitoring)       │ │
        │  └──────────────────────────────────────┘ │
        └──────────┬────────────────────────────────┘
                   │
     ┌─────────────┼──────────────┐
     │             │              │
     ▼             ▼              ▼
┌─────────┐  ┌─────────┐   ┌─────────┐
│ Secondary│  │Secondary│  │  Tertiary│
│  AG Node │  │ Replica │  │ (Optional)│
│ (Async)  │  │(Repl.)  │  │ Log Ship │
└─────────┘  └─────────┘   └─────────┘
     │             │              │
     └─────────────┼──────────────┘
                   │
              [BACKUP VAULT]
        (Long-term retention,
         Compliance storage)
```

**Key Integration Points for Senior DBAs:**

- **Connection routing** through AG Listener ensures transparent failover to applications
- **Security layers** (TDE, Always Encrypted, Audit, RLS) span from storage to query time
- **Maintenance jobs** must account for replicated databases (secondary replicas may not participate in index maintenance)
- **Monitoring & Tracing** requires DMV queries, Extended Events, and health checks across all nodes
- **Backup strategy** encompasses full backups, differential backups, log backups, and their replication to secondary sites

---

## Foundational Concepts

### Key Terminology

**RPO (Recovery Point Objective)**
- Maximum acceptable **data loss** measured in time
- Example: RPO = 5 minutes means losing up to 5 minutes of transactions is acceptable
- Driven by business requirements and cost/complexity trade-offs
- Synchronous replication achieves RPO ≈ 0 (no data loss); asynchronous replication RPO = recovery lag time

**RTO (Recovery Time Objective)**
- Maximum acceptable **downtime** before the system must be operational
- Example: RTO = 15 minutes means users must regain access within 15 minutes
- Automatic failover (AG) achieves RTO in seconds; manual processes may require hours
- RTO must be carefully tested and validated in disaster simulations

**SLA (Service Level Agreement)**
- Contractual promise of uptime percentage (99.9%, 99.99%, 99.999%)
- 99.9% = 8.76 hours downtime/year (43.8 minutes/month)
- 99.99% = 52.6 minutes/year (4.38 seconds/month)
- Financial penalties or credits apply if SLA is breached
- Must be achievable given infrastructure, people, and processes

**RFO (Recovery Failure Objective)**
- Probability of recovery failing when needed
- Senior DBAs must design and test for RFO close to 0%
- Involves redundancy, automation, and regular failover testing

**MTBF (Mean Time Between Failures)**
- Statistical average time before a component fails
- Influences how frequently disasters should be simulated
- High-availability designs aim to reduce customer-visible MTBFs by adding redundancy

**MTTR (Mean Time To Repair)**
- Average time to restore failed component to service
- Includes detection time, diagnosis time, and actual repair
- Senior DBAs optimize this through automation, documentation, and playbooks

---

### Database Architecture Fundamentals

#### Transaction Processing Fundamentals

At the core of SQL Server operations is the **ACID transaction model**:

**Atomicity**
- Transaction succeeds completely or fails completely; no half-finished updates
- Write-Ahead Logging (WAL) ensures consistency

**Consistency**
- Database moves from one valid state to another; constraints are preserved
- Referential integrity, check constraints, and application logic maintain consistency

**Isolation**
- Concurrent transactions don't interfere with each other
- SQL Server supports four isolation levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable
- Snapshot Isolation (SI) provides optimistic concurrency for many workloads

**Durability**
- Committed transactions survive system failures
- Transactions are written to transaction log before commit acknowledgment
- Log files must be on reliable storage (RAID-protected; cloud-managed storage with durability guarantees)

#### Data Replication Models

**Synchronous Replication** (Availability Groups)
- Primary waits for secondary acknowledgment before committing
- Ensures zero data loss (RPO = 0)
- Performance trade-off: latency increases with distance/network latency
- Suitable for same-datacenter HA or closely located regional DR

**Asynchronous Replication** (Transactional Replication, Replication, Log Shipping)
- Primary commits without waiting for secondary acknowledgment
- Secondary "catches up" independently
- Lower performance impact; acceptable data loss (RPO = lag time)
- Suitable for DR across distant regions or read-scale scenarios

**Snapshot-Based Replication** (Point-in-time consistency)
- Database is captured at a specific moment
- Used for initial seeding or periodic full updates
- No ongoing incremental synchronization

---

#### Transaction Log Architecture

The transaction log is the **heartbeat** of SQL Server; it enables:

- **Atomicity**: Ensures all-or-nothing transaction outcomes
- **Durability**: Persists committed data before acknowledging to applications
- **Replication**: Captures all changes for distribution to secondaries
- **Recovery**: Replays log after crashes to restore committed transactions
- **Backup/Restore**: Enables point-in-time recovery

**Log Structure:**
- Written sequentially (extremely fast: sequential I/O optimization)
- Divided into virtual log files (VLFs); DBAs monitor VLF fragmentation
- LSN (Log Sequence Number): unique identifier for each log record
- Checkpoint: periodic marker indicating which log records are durable in the data file

**Best Practice:**
- Log file on fast, dedicated storage (NVMe or high-speed SSD)
- Log should *never* be on same spindle as data files (contention → deadlocks)
- Monitor log growth; excessive growth indicates slow readers (replication lag, long transactions)

---

#### Backup & Recovery Strategy

**Full Backup**
- Captures entire database at a point in time
- Largest backup; slowest to take and restore
- Essential baseline for any recovery strategy

**Differential Backup**
- Captures only blocks modified *since last full backup*
- Significantly smaller than full; faster to take
- Restore requires: full backup + last differential backup

**Transaction Log Backup**
- Captures committed transactions since last log backup
- Smallest backups; enables point-in-time recovery (PITR)
- Truncates log to prevent unbounded growth
- Essential for production databases

**Copy-Only Backup**
- Special backup that doesn't affect backup chain
- Used for point-in-time snapshots without breaking full/diff/log sequence
- Useful for testing or one-off requirements

**Recovery Models:**

| **Model** | **Purpose** | **Log Behavior** | **RPO Possible** |
|-----------|-----------|---------|---------|
| Full | Production; supports PITR | Logs all changes; requires log backups | Hours (log backup frequency) |
| Bulk-logged | High-volume operations (bulk inserts, index builds) | Logs minimally during bulk ops | Minutes |
| Simple | Development/testing only | Auto-truncates; doesn't persist for replication | Data loss possible since last backup |

---

### Important DBA Principles

#### 1. **Defense in Depth (Security)**
Never rely on a single security layer. Implement overlapping controls:
- Network isolation (firewalls, VPCs)
- Authentication (Windows, SQL Auth, MFA)
- Authorization (least-privilege roles)
- Encryption (at-rest via TDE, in-transit via TLS, in-use via Always Encrypted)
- Auditing (who did what, when, where)
- Row-level security for data compartmentalization

#### 2. **Test Everything in Non-Production First**
- Failover procedures must be practiced before disasters strike
- Backup/restore procedures *must* be tested; backups are only valid if verified
- Schema changes and maintenance scripts must be validated on test replicas
- Senior DBAs allocate 10-15% of time to chaos engineering and disaster simulations

#### 3. **Automate Routine Operations**
- Manual operations = human errors + inconsistency
- SQL Server Agent jobs enable scheduled execution of:
  - Index maintenance (rebuild/reorganize)
  - Statistics updates
  - Backup jobs
  - Health checks and alerting
  - Log shipping restore
- Infrastructure-as-Code (IaC) should define backup policies, retention, replication settings

#### 4. **Monitor Proactively, React Reactively**
- Monitoring is not optional; it's foundational
- Alerting must be tuned to avoid alert fatigue (too many false alarms = ignored alerts)
- DMVs provide deep insights into:
  - Query performance (`sys.dm_exec_query_stats`)
  - Wait statistics (`sys.dm_os_wait_stats`)
  - Active transactions (`sys.dm_tran_active_transactions`)
  - Blocking (`sys.dm_tran_locks`)
- Extended Events capture detailed diagnostic data when needed

#### 5. **Document and Communicate**
- Production architecture diagrams: what connects to what, and why
- Runbooks: step-by-step procedures for common operations (failover, restore, etc.)
- Change log: track all infrastructure modifications
- Disaster recovery procedures must be *documented, tested, and accessible*

#### 6. **Plan for Failure**
- Assume hardware will fail; design for it
- Assume networks will partition; have a quorum strategy
- Assume disks will fill up; monitor carefully and alert early
- Assume humans will make mistakes; implement code reviews and approvals for production changes

---

### Best Practices by Category

#### Availability & Disaster Recovery

| **Best Practice** | **Rationale** |
|---|---|
| Define RPO/RTO explicitly | Without clear targets, you'll build the wrong solution |
| Test failover quarterly (minimum) | Untested failover = likely to fail when needed |
| Use synchronous AG for same-datacenter HA | Zero data loss + automatic failover = best user experience |
| Use asynchronous replication for geographic DR | Distant replication = lower performance impact |
| Implement automated failover where possible | Manual failover = delayed recovery + human error |
| Maintain quorum in odd-numbered AG nodes | Prevents split-brain; ensures only one primary active |
| Monitor AG health continuously | Don't wait for users to report availability issues |
| Document failover procedure and expected behavior | When disaster strikes, dry-run procedures in playbooks |

#### Security Best Practices

| **Best Practice** | **Rationale** |
|---|---|
| Enforce Windows Authentication where possible | Easier to manage, audit, and integrate with enterprise identity |
| Apply least-privilege principle strictly | Minimize attack surface; users should have only required permissions |
| Enable TDE for sensitive databases | Protects data at rest; mandatory for compliance (HIPAA, PCI-DSS, GDPR) |
| Use Always Encrypted for columns with PII/PHI | Even if DB is compromised, encrypted columns remain protected |
| Implement Row-level Security for multi-tenant isolation | Prevent cross-tenant data leakage in shared databases |
| Enable audit and maintain audit logs securely | Enables forensic investigation and regulatory compliance |
| Use strong passwords and rotate service accounts periodically | Default/weak credentials are #1 attack vector |
| Restrict server-level role membership | sa account should not be used for regular connections |
| Isolate database servers on dedicated network | Segment from web tier, avoid direct internet exposure |

#### Maintenance Best Practices

| **Best Practice** | **Rationale** |
|---|---|
| Perform index maintenance weekly/bi-weekly (REBUILD/REORGANIZE) | Fragmentation degrades query performance by 20-70% over time |
| Update statistics after bulk operations | Stale statistics → bad query plans → poor performance |
| Verify database integrity weekly (DBCC CHECKDB) | Detect corruption early before it spreads |
| Maintain transaction log at healthy size | Oversized logs consume disk, slow backup/restore |
| Back up transaction logs frequently (every 1-15 minutes in high-volume DBs) | Minimizes RPO; enables point-in-time recovery |
| Test restore procedures monthly | Backups are only valid if verified; restore testing proves this |
| Clean up old backups per retention policy | Disk costs; regulatory compliance; operational efficiency |
| Schedule maintenance during low-usage windows | Avoid impacting business operations |

#### Monitoring Best Practices

| **Best Practice** | **Rationale** |
|---|---|
| Establish baseline metrics for normal operations | Anomalies are only detectable with known baseline |
| Monitor CPU, memory, disk I/O, and network utilization | Hardware constraints bottleneck query performance |
| Alert on replication lag (AG, snapshot, transactional) | Lagging replicas = stale data, violate RPO |
| Track AG sync state; alert on async drift | Automatic failover only works if replicas are synchronized |
| Monitor transaction log utilization | Prevent log-full scenarios that block operations |
| Set up deadlock alerts and capture deadlock graphs | Deadlocks waste resources; graphs show root cause |
| Audit failed login attempts | Potential intrusion attempts or misconfigured connections |
| Review SQL Server error log daily | Errors revealed early enable proactive fixes |

---

### Common Misunderstandings & Clarifications

#### Misunderstanding #1: "Availability Groups = Backup Solution"

**Reality:**
- AGs replicate data for HA/DR; they do *not* protect against logical corruption or accidental deletion
- If a stored procedure accidentally deletes all customer records, the deletion propagates to all AG replicas instantly
- **Solution**: Maintain regular backups AND use Availability Groups

#### Misunderstanding #2: "Replication Ensures Zero Data Loss"

**Reality:**
- Asynchronous replication (Transactional Replication, Log Shipping) causes data loss if primary fails during lag
- Only synchronous replication (AG with synchronous commit) guarantees zero data loss
- **Best Practice**: Define RPO and choose the replication strategy matching that RPO

#### Misunderstanding #3: "TDE Encrypts Everything"

**Reality:**
- TDE encrypts data at rest (on disk); it does *not* encrypt data in memory or in transit
- If someone connects to SQL Server and queries, they see unencypted data on-screen
- **Solution**: Combine TDE (at-rest) with Always Encrypted (in-use) and TLS (in-transit) for complete protection

#### Misunderstanding #4: "Row-level Security = Complete Multi-Tenancy Isolation"

**Reality:**
- RLS filters rows returned by SELECT; it doesn't prevent inference attacks (sum queries, timing attacks)
- RLS is an application-level control; determined by security predicates evaluated at query time
- A single SQL injection vulnerability bypasses RLS
- **Best Practice**: RLS is *one layer*; combine with encryption, application validation, and audit logging

#### Misunderstanding #5: "DMVs Show Historical Data"

**Reality:**
- DMVs (`sys.dm_*`) show current state since last service restart
- They do *not* retain historical data; restarting SQL Server clears DMV data
- Extended Events and trace files persist data to disk and survive restarts
- **Best Practice**: Implement continuous collection (Extended Events, monitoring agent) if long-term analysis is needed

#### Misunderstanding #6: "Synchronous AG = No Performance Impact"

**Reality:**
- Synchronous AG incurs latency = network round-trip time to secondary + log hardening time on secondary
- Over long-distance networks (50+ ms), this can noticeably increase query latency
- **Best Practice**: Keep synchronous AGs within same datacenter; use asynchronous for geographic DR

#### Misunderstanding #7: "Automated Failover = No Downtime"

**Reality:**
- Failover is fast (seconds to tens of seconds) but not instantaneous
- Applications using connection pooling may cache old primary DNS; they must reconnect
- Some in-flight queries may be lost during failover
- **Best Practice**: Test failover impact on applications; design apps to retry failed connections

#### Misunderstanding #8: "Monitoring = Alert on Every Metric"

**Reality:**
- Too many alerts = alert fatigue = ignored alerts when real problems occur
- Alert thresholds must be tuned to normal operational variance
- **Best Practice**: Alert on anomalies and SLA violations; use dashboards for trending

#### Misunderstanding #9: "Full Backups Are Always Needed"

**Reality:**
- Full backups are *baseline* for a backup chain; differential maintains the chain
- In some scenarios (immense databases), differential + log backups may be sufficient
- Recovery time increases if you must restore full + multiple differentials + log backups
- **Best Practice**: Right-size backup strategy: full (weekly) + diff (daily) + log (every 15 min for production)

#### Misunderstanding #10: "Statistics Are Only Needed After Index Maintenance"

**Reality:**
- Statistics degrade continuously as data changes
- Missing statistics (unused indexes) and outdated statistics both cause bad query plans
- Auto-Update Statistics option works well for online OLTP but may lag in data warehouses
- **Best Practice**: In DW environments, schedule explicit `UPDATE STATISTICS` jobs; monitor plan quality with DMVs

---

---

## HA/DR Fundamentals

### Textual Deep Dive

#### Internal Working Mechanism

High Availability and Disaster Recovery operate on fundamentally different paradigms, though both utilize redundancy:

**High Availability (HA)** focuses on *local resilience* within a single datacenter or closely-located facilities:
- **Objective**: Minimize downtime from planned maintenance or unplanned component failures
- **Scope**: Same-site redundancy (multiple SQL Server instances, shared storage via SAN/cluster, or replicated storage)
- **Failover Time**: Seconds to minutes (fast, automatic)
- **RPO**: Typically 0 (synchronous replication)
- **Typical Configuration**: Failover Cluster Instance (FCI) or Availability Groups with synchronous commit

**Disaster Recovery (DR)** focuses on *geographic resilience* against site-wide failures:
- **Objective**: Recover operations at an alternate location after catastrophic primary site failure
- **Scope**: Geographic separation (often hundreds of miles or more)
- **Failover Time**: Minutes to hours (often manual or semi-automatic)
- **RPO**: Typically minutes to hours (asynchronous replication acceptable)
- **Typical Configuration**: Availability Groups (async), Replication, or Log Shipping to secondary site

#### Design Decision Matrix for HA/DR Strategy

The choice of HA/DR mechanism depends on trade-offs across multiple dimensions:

| **Dimension** | **Failover Cluster (FCI)** | **Availability Groups (Sync)** | **Availability Groups (Async)** | **Replication** | **Log Shipping** |
|---|---|---|---|---|---|
| **RPO** | 0 | 0 | Seconds-Minutes | Minutes-Hours | Hours |
| **RTO** | 1-5 min | 10-30 sec | 10-30 sec | Hours | Hours |
| **Failover Type** | Automatic | Automatic | Automatic | Manual | Manual |
| **Complexity** | Moderate | Moderate-High | Moderate-High | High | Low |
| **Cost** | High (SAN) | Moderate (licenses + networking) | Moderate | Moderate | Low |
| **Geographic Span** | Same datacenter | Same or nearby | Region-wide | Global | Global |
| **Read Scaling** | No | Secondary read-only | Secondary read-only | Yes | Limited (recovery) |
| **Data Loss Allowed** | None | None | Acceptable | Acceptable | Acceptable |
| **Production Readiness** | Immediate | Immediate | Immediate | Minutes-Hours | Hours |

**Key Decision Flow:**

```
┌─ Is Zero Data Loss Required? ─┐
│                               │
├─ YES ──────────────────────→ Synchronous AG (same/nearby datacenter)
│                               or Failover Cluster
│
└─ NO ───────────────────────→ ┌─ Is RTO < 30 minutes Required? ─┐
                               │                                   │
                               ├─ YES ─→ Asynchronous AG (regional)
                               │
                               └─ NO ──→ ┌─ Read Scaling Needed? ─┐
                                         │                         │
                                         ├─ YES ─→ Replication
                                         │
                                         └─ NO ──→ Log Shipping
```

#### Production Usage Patterns

**Pattern 1: Hybrid HA/DR (Tier-1 Mission-Critical)**
```
Data Center A (Primary)          Data Center B (Regional)     Data Center C (OFFSITE)
└─ AG Sync Primary               └─ AG Sync Secondary         └─ Log Shipping Warm Standby
   (Synchronous replication)        (Auto-failover)             (Manual activation)
        ↓                             ↓
        └──────────────────────────────┘
                Replicated every ~1 sec
                Data Loss: 0 seconds
                Failover: 10-30 seconds (auto)
```

**Pattern 2: Read-Scale + Regional DR (Analytics Workload)**
```
Data Center A                    Data Center B                Data Center C (Cloud)
└─ AG Primary (OLTP)            └─ AG Async (Read-Only)       └─ Replication Subscriber
   Synchronous with B              Reports/Analytics            (Data Warehouse)
   ↓                               ↓
   Every transactionreplicates   Accepts read queries          Async replication
   in milliseconds               (not real-time current)       for historical analysis
```

**Pattern 3: Low-Cost Warm Standby (Non-Production Databases)**
```
Data Center A                    DR Site (Geographic)
└─ Production Database           └─ Standby Database
   Logs backed up hourly            Restored from log backup hourly
   ↓                               ↓
   Backup → Backup Location → Log Shipping Agent → Standby
   ↓
   RPO: 1 hour | RTO: 1-2 hours (manual)
```

#### DBA Best Practices for HA/DR Strategy

1. **Establish Clear Business Requirements First**
   - Never design HA/DR in technical vacuum
   - Interview business stakeholders: what is acceptable downtime? data loss?
   - Document SLA, RPO, RTO explicitly in design document

2. **Right-Size Your Solution**
   - Avoid over-engineering (synchronous AG everywhere = high cost, network overhead)
   - Avoid under-engineering (log shipping for Tier-1 app = unacceptable RTO)
   - Balance cost vs. risk tolerance

3. **Design for Graceful Degradation**
   - When primary fails, secondary becomes primary (or standby activates)
   - Applications must handle connection failover (idle connection timeout + retry)
   - Consider partial failures (primary accessible but degraded)

4. **Test Failover Procedures Quarterly (Minimum)**
   - Schedule **planned failover drills** during maintenance windows
   - Document expected behavior during failover (what breaks, how long)
   - Validate RTO/RPO measurements from real failovers, not estimates
   - Automate failover testing where possible

5. **Monitor HA/DR Health Continuously**
   - AG synchronization state (synchronized vs. not synchronized)
   - Replication lag (subscriber LSN vs. publisher LSN)
   - Log shipping restore lag (standby LSN vs. primary LSN)
   - Alerting on drift prevents failures from going unnoticed

6. **Document Runbooks and Make Them Accessible**
   - Step-by-step failover procedure (automated or manual)
   - Rollback procedure (fail back to original primary)
   - Contact information for escalation
   - Store in version control; ensure accessibility during emergencies

#### Common Pitfalls and How to Avoid Them

| **Pitfall** | **Consequence** | **Avoidance Strategy** |
|---|---|---|
| **No Failover Testing** | RTO/RPO estimates are fantasies; actual failover fails or runs 10x slower | Schedule quarterly failover tests; measure actual time; update runbooks |
| **Accepting Estimated RTO/RPO** | When disaster strikes, recovery takes far longer than promised to business | Validate estimates with real failover; include network latency, app reconnect time |
| **Over-Synchronous Replication** | High latency on all queries; business complains about app responsiveness | Use synchronous only for HA (same datacenter); use async for distant DR |
| **Forgetting Backup Strategy** | HA/DR protects against **failures**; backups protect against **logical corruption** | Maintain independent backup schedule even with HA/DR |
| **Not Monitoring Replication Lag** | Secondary falls behind unnoticed; when failover occurs, significant data loss | Monitor replication/shipping lag; alert if lag exceeds RPO threshold |
| **Quorum Misconfiguration** | Split-brain scenario; both primaries think they are primary; data corruption | Understand quorum requirements; odd number of nodes for automatic failover |
| **Network Partitions Unplanned** | Failover activates due to network glitch (not real failure); users on both sides confused | Implement health checks; network monitoring; clear escalation procedure |
| **Failing Over Without Draining** | In-flight transactions lost; applications see connection reset | Implement connection drain before failover; application-level retry logic |
| **Not Accounting for Licensing** | Passive standby replicas incur licensing cost (SQL Server requires CAL per connection) | Verify SQL Server licensing model (Standard vs. Enterprise) affects HA/DR choice |

---

## Always On Availability Groups

### Textual Deep Dive

#### Internal Working Mechanism

**Availability Groups** are SQL Server's premier HA/DR solution, providing automated failover with optional read-only secondary replicas. They operate at the database level (not instance level) and leverage the transaction log as the replication mechanism.

##### Architecture Components

```
 ┌─────────────────────────────────────────────────────────────┐
 │                    AVAILABILITY GROUP                       │
 │                  (Logical Container)                        │
 └────────────────────┬─────────────────────────────────────────┘
                      │
    ┌─────────────────┼─────────────────┐
    │                 │                 │
    ▼                 ▼                 ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│   PRIMARY   │  │ SECONDARY 1  │  │ SECONDARY 2  │
│  (Read/Wrt) │  │ (RO or sync) │  │  (RO + async)│
│ Instance A  │  │  Instance B  │  │  Instance C  │
│ SQL Server  │  │  SQL Server  │  │  SQL Server  │
└─────────────┘  └──────────────┘  └──────────────┘
     │                │                  │
     └────────────────┼──────────────────┘
                      │
            ┌─────────▼─────────┐
            │  AG LISTENER      │
            │  (Virtual IP/DNS) │
            │  AG_Listener_IP   │
            │  Port: 1433       │
            └───────────────────┘
               │
               ▼
        Applications Connect
        (transparent failover)
```

**Key Components:**

1. **Primary Replica**
   - Only replica accepting read/write operations
   - Source of truth for data
   - Generates transaction log records
   - Sends log blocks to secondary replicas

2. **Secondary Replicas**
   - Receive log blocks from primary
   - Replicate to their own transaction log
   - Can be configured as read-only (for reporting, backups)
   - Can be synchronous or asynchronous

3. **Availability Group Listener**
   - Virtual network name (DNS alias) pointing to currently active primary
   - Applications connect to listener (not directly to instance)
   - Failover redirects listener to new primary; applications reconnect automatically
   - Multiple listeners can be created (for read-only routing to secondaries)

4. **Quorum**
   - Cluster voting mechanism ensuring only one primary is active (prevents split-brain)
   - Node count determines quorum model

##### Synchronous vs. Asynchronous Commit

**Synchronous Commit Mode:**
```
Client sends INSERT command
    ↓
Primary receives and writes to log
    ↓
Primary sends log block to secondary(ies)
    ↓
Primary WAITS for secondary(ies) to acknowledge:
"I have received and written this log block"
    ↓
When acknowledgment received from ALL secondary replicas:
    ├─ Primary commits the transaction
    ├─ Sends confirmation to client: "Your transaction succeeded"
    └─ ✓ RPO = 0 (zero data loss)

If secondary doesn't acknowledge within timeout:
    ├─ Primary can be configured to FAIL the transaction
    │  (Primary Availability Group Role = ALL MANDATORY)
    │  or
    └─ Allow primary to continue (graceful degradation)
       (Primary Availability Group Role = ALLOW_UNSYNCED_COMMIT)
```

**Asynchronous Mode:**
```
Client sends INSERT command
    ↓
Primary receives and writes to log
    ↓
Primary sends log block to secondary(ies)
    ↓
Primary immediately commits WITHOUT waiting:
    ├─ Sends confirmation to client: "Your transaction succeeded"
    │
    └─ Secondary(ies) receive log block in background (no hurry)
       └─ Write to their log and replicate

If primary fails in next millisecond:
    ├─ Secondary(ies) might not have received the log block yet
    └─ ✗ RPO = Replication lag (data loss)
       Typical lag: 1 second to several seconds
```

**Quorum Models and Failover:**

| **Node Count** | **Quorum Type** | **Tolerable Failures** | **Failover Behavior** |
|---|---|---|---|
| 3 nodes (1 primary + 2 secondary) | Node Majority | 1 failure → continue; 2 failures → stop | Automatic to 1st secondary if in quorum |
| 5 nodes (1 primary + 4 secondary) | Node Majority | 2 failures → continue; 3+ failures → stop | Automatic with flexibility to choose failover target |
| 2 nodes + File Share Witness | Node + File Share | 1 node + witness failure → continue | Automatic when primary fails and secondary + witness in quorum |
| 2 nodes + Cloud Witness (Azure) | Node + Cloud | 1 node + cloud witness failure → continue | Automatic; witness in Azure storage |

**Important**: With only 2 nodes and NO witness, neither can claim quorum alone. If network partitions, both think they should be primary → split-brain. **Never deploy 2-node AG without witness.**

#### Production Usage Patterns

**Pattern 1: Same-Datacenter HA (Local Resilience)**
```
Building A (Datacenter 1)
├─ Primary Replica (Server 1)
├─ Secondary Replica 1 (Server 2) - Synchronous Commit
└─ Secondary Replica 2 (Server 3) - Synchronous Commit

Network: High-speed backend network (< 1ms latency, gigabit)
Quorum: Node Majority (3 nodes)
RPO: 0 (zero data loss)
RTO: 10-30 seconds (automatic failover)
Use Case: Server maintenance, hardware failures, planned restarts
```

**Pattern 2: Regional DR (Geo-Distributed Resilience)**
```
Datacenter A (Primary Region)
├─ Server 1: Primary (Synchronous Commit)
├─ Server 2: Secondary (Synchronous Commit)
└─ Network: < 1ms latency

        ↓ (WAN link, managed Azure ExpressRoute)

Datacenter B (Secondary Region)
└─ Server 3: Secondary (Asynchronous Commit)
   Network: 50-100ms latency
   
RPO: 0 for local nodes; acceptable lag (seconds) to DR site
RTO: < 1 minute for local failover; 5-10 minutes for DR site activation
Quorum: Node Majority (3 nodes) or Ext. Cluster Quorum if > 3
Use Case: Entire datacenter failure; regional disaster
```

**Pattern 3: Read-Scale + DR (Hybrid)**
```
Primary (OLTP Workload)
├─ Server 1: Read/Write

Secondary Replicas
├─ Server 2: Synchronous, Read-Only (local reporting)
├─ Server 3: Asynchronous, Read-Only (DR site reporting)

Listeners Created:
├─ AG_Primary: Routes to Server 1 (OLTP queries)
└─ AG_Secondary: Routes to Server 2 or 3 (read-only queries)

Benefit: Distribute read workload; free up primary for write-heavy OLTP
```

#### DBA Best Practices for AG Configuration

1. **Choose Synchronous Carefully**
   - Synchronous = latency increase proportional to network round-trip time
   - Rule of Thumb: Synchronous if latency < 10ms; async if > 50ms
   - Monitor primary transaction latency after AG failover group change

2. **Architect Quorum Correctly**
   - 1 Primary + 2 Secondaries = 3 nodes → Node Majority quorum
   - If adding witness, use odd numbers: (1P + 1S) + external witness, or (1P + 2S) no witness
   - Test quorum behavior: isolate nodes and verify primary continues appropriately

3. **Use Read-Intent Routing for Read-Only Secondaries**
   - Configure secondary replicas as "read-only"
   - Create separate read-only listener
   - Application specifies `ApplicationIntent=ReadOnly` in connection string
   - Directs query to secondary automatically (reduces primary load)

4. **Plan for Listener Failover**
   - Listener DNS name must resolve to available primary
   - DNS cache may interfere: set application connection timeout short (< 10 seconds)
   - Applications must implement retry logic on connection failure
   - Test listener failover; don't assume automatic

5. **Monitor AG Health Proactively**
   - **Synchronization Health**: `sys.dm_hadr_group_hadrstate`
   - **Replica Details**: `sys.dm_hadr_database_replica_states`
   - **Availability Replica Health**: Monitor Cluster Name Objects (CNOs) in cluster
   - Alert if secondary becomes NOT SYNCHRONIZED unexpectedly

6. **Plan Backup Strategy Carefully**
   - Backups CAN be taken from secondary replicas (reducing primary load)
   - But full database backup must always start from primary
   - Log backups can be taken from any replica (if configured)
   - Track backup LSN to ensure backup chain is valid

#### Common Pitfalls for Always On AG

| **Pitfall** | **Symptom** | **Root Cause** | **Fix** |
|---|---|---|---|
| **Listener Failover Delays** | Users experience 30+ second timeout after failover | DNS caching; app doesn't retry quickly | Set app connection timeout < 10 sec; invalidate DNS cache post-failover |
| **Secondary Unsynchronized** | Replica shows "NOT SYNCHRONIZED" even though network is healthy | Secondary lagging due to slow log apply; network lag; resource bottleneck | Check secondary for CPU/disk bottleneck; review network latency; increase secondary resources |
| **Split-Brain Scenario** | Two primaries claim to be primary; data corruption | Network partition; 2-node AG without witness | Add witness immediately; troubleshoot network; failover manually to known-good primary |
| **Backup Fails After Failover** | Backup job fails on new primary | Backup job configured for old server name (not listener) | Configure backup jobs to use AG listener name; test after failover |
| **Long Initial Sync** | Secondary takes days to synchronize | Full database > 1TB; network bandwidth limited; secondary is slow | Use backup + restore for initial seeding instead of log-based seeding; verify network capacity |
| **Synchronous AG with High Latency** | Applications experience query timeouts after AG failover | Latency > 100ms; synchronous mode adds roundtrip delay; queries slower than before | Switch to asynchronous for distant replicas; monitor latency; keep synchronous local only |
| **Quorum Missing** | Cluster goes offline; can't failover | Quorum node failure undetected; no monitoring | Implement quorum monitoring; understand majority rules; test quorum scenarios quarterly |
| **Listener Not Responding** | Listener DNS resolves but connection fails | Listener virtual IP not created properly; firewall blocking; AG offline | Verify listener IP; test listener connectivity from app server; check cluster health |

---

## Replication

### Textual Deep Dive

#### Internal Working Mechanism

SQL Server Replication is an asynchronous publish-subscribe model for distributing data changes across instances and databases. Unlike Availability Groups (which replicate entire databases), Replication can be configured for:

- **Snapshot**: One-time or periodic full copy of data
- **Transactional**: Incremental replication of committed transactions
- **Merge**: Bi-directional replication with conflict resolution

##### Replication Architecture

```
┌────────────────────────────────────────────────────────────┐
│              PUBLISHER (Source Database)                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Publication: Articles (Tables, Views, Functions)    │  │
│  │ ├─ Article 1: Customers (All Columns)              │  │
│  │ ├─ Article 2: Orders (Filtered by Region)          │  │
│  │ └─ Article 3: view_CustomerOrderSummary            │  │
│  │                                                      │  │
│  │ Transaction Log (Replication Marked)                │  │
│  │ ├─ Each committed transaction marked               │  │
│  │ └─ LSN (Log Sequence Number) tracks progress       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────┬─────────────────────────────────────────────────────┘
         │
         │ (SQL Server Agent Job)
         │ Log Reader Agent (Transactional Replication)
         │ or Snapshot Agent (Snapshot Replication)
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│          DISTRIBUTOR (Distribution Database)                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Distribution Database:                               │   │
│  │ ├─ MSrepl_transactions (transaction metadata)        │   │
│  │ ├─ MSrepl_commands (T-SQL commands to apply)         │   │
│  │ └─ Subscription queues                               │   │
│  │                                                       │   │
│  │ SQL Agent Jobs:                                       │   │
│  │ ├─ Log Reader Agent (reads from publisher log)       │   │
│  │ ├─ Snapshot Agent (generates snapshots)              │   │
│  │ └─ Distribution Agent (pushes to subscribers)        │   │
│  └──────────────────────────────────────────────────────┘   │
└────┬──────────────────────────────────────────────────────────┘
     │ (Push or Pull)
     ▼
┌─────────────────────────────────────────────────────────────┐
│         SUBSCRIBER (Receiving Database)                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ Subscription: Replicated Articles                    │   │
│ │ ├─ Article 1 Copy: Customers table                  │   │
│ │ ├─ Article 2 Copy: Orders (filtered subset)         │   │
│ │ └─ Article 3 Copy: view_CustomerOrderSummary        │   │
│ │                                                      │   │
│ │ SQL Agent Job (Pull):                               │   │
│ │ └─ Distribution Agent (fetches changes from dist)   │   │
│ │    or Merge Agent (Merge Replication)              │   │
│ │                                                      │   │
│ │ LSN/Watermark Track Progress                       │   │
│ │ └─ Prevents re-applying same change multiple times │   │
│ └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

##### Replication Types

**1. Snapshot Replication**

Mechanism: Publisher generates complete copy of publishing database at snapshot time; Distributor stores snapshot; Subscriber downloads and applies.

```
Trigger Event (Schedule or Manual)
    ↓
Snapshot Agent (on Distributor or Publisher)
    ├─ Locks published tables briefly (exclusive lock during snapshot)
    ├─ Generates BCP output files (native format)
    └─ Stores snapshot in \\DistributorServer\REPDATA\unc (or local path)
    ↓
Snapshot Available
    ↓
Distribution Agent (on Subscriber, if pull) or Distributor (if push)
    ├─ Downloads snapshot files
    ├─ Creates schema if doesn't exist (or drops existing if specified)
    └─ Bulk-inserts all data
    ↓
Subscription Synchronized

RPO: Limited by snapshot interval (hourly, daily, etc.)
RTO: Minutes to hours (depending on dataset size)
Use Case:
    ├─ Initial subscription setup
    ├─ Data warehouse loads (complete refresh)
    ├─ Small, read-heavy datasets
    └─ Non-continuous sync requirements
```

**Pros:**
- Simple to understand and troubleshoot
- Suitable for large dataset initialization
- Read-only subscribers (no conflicts)

**Cons:**
- Entire dataset re-sent on snap replication (inefficient for large tables)
- Snapshot generation locks tables (production impact)
- Not suitable for frequent updates

**2. Transactional Replication**

Mechanism: Publisher streams individual transactions to Distributor; Distributor applies to Subscriber sequentially.

```
Client executes on Publisher:
    INSERT INTO dbo.Orders VALUES (...)
    INSERT INTO dbo.OrderDetails VALUES (...)
    COMMIT;
    ↓
Transaction committed to Publisher Log
    ↓
Log Reader Agent (SQL Agent Job)
    ├─ Scans Publisher transaction log
    ├─ Identifies changes (INSERT, UPDATE, DELETE)
    ├─ Forwards to Distribution database as commands
    │  (INSERT command, UPDATE command, etc.)
    └─ Marks transaction as replicated (LSN)
    ↓
Distributor stores commands in MSrepl_commands table
    ↓
Distribution Agent (Push or Pull)
    ├─ Retrieves commands from Distributor
    ├─ Applies to Subscriber in same order
    └─ Maintains LSN to ensure no skips
    ↓
Subscriber reflects changes

RPO: 1-10 seconds (lag between publisher and subscriber)
RTO: Seconds (failover to subscriber)
Use Case:
    ├─ Real-time data distribution (Sales, Inventory)
    ├─ Read-only reporting subscribers
    ├─ Cascading replication (A→B→C)
    └─ High-volume OLTP environments
```

**Pros:**
- Efficient bandwidth usage (only changes sent)
- Near real-time synchronization
- Read-only subscribers (no conflicts)
- Cascading replication supported (daisy-chain multiple subscribers)

**Cons:**
- Higher complexity than snapshot
- Log Reader bottleneck if many changes
- Replication lag possible if network slow

**3. Merge Replication**

Mechanism: Bi-directional replication; both Publisher and Subscriber can modify data; conflict resolution applied based on policy.

```
Publisher                              Subscriber
├─ INSERT row with ID=100             ├─ INSERT row with ID=100
│  (different data)                    │  (different data)
│                                      │
└─ MERGE Agent triggers               ├─ MERGE Agent triggers
   │                                    │
   └────────────── Compare ────────────┘
        │
        ▼
    Conflict Detected:
    ├─ Publisher last-write-wins?
    ├─ Subscriber last-write-wins?
    ├─ Priority-based (custom logic)?
    └─ Manual resolution required?

RPO: Variable (depends on sync schedule and reconciliation)
RTO: Minutes to hours (manual resolution possible)
Use Case:
    ├─ Mobile/disconnected scenarios (laptops, mobile devices)
    ├─ Branch office data collection (collect locally, merge at central)
    ├─ Multi-site updates requiring conflict resolution
    └─ Occasional connectivity (dial-up, WAN intermittency)

Conflict Resolution Policies:
    ├─ Publisher Wins (most common; centralized is source of truth)
    ├─ Subscriber Wins (rare; subscriber data takes precedence)
    ├─ Custom Resolver (invoke CLR function or script for logic)
    └─ Manual (DBA reviews and picks winner)
```

**Pros:**
- True bi-directional replication
- Suitable for disconnected scenarios
- Flexible conflict resolution policies

**Cons:**
- Most complex to configure and troubleshoot
- Conflict resolution overhead
- Higher licensing cost in some scenarios
- Risk of inconsistency if poorly designed

#### Production Usage Patterns

**Pattern 1: Transactional Replication for Reporting**
```
OLTP Database (Publisher)          Reporting Subscriber
├─ 1000 transactions/sec           ├─ Read-only copy
├─ 24/7 high-volume updates        ├─ Lagged replication (2-5 min)
│                                  │  (allows reports to run without
│ Log Reader Agent                 │   blocking production queries)
│ ↓                                │
│ Distributor                      │ Distribution Agent
│ ↓                                │
└─────────────────────────────────┘

Benefits:
    ├─ Offload reporting from production server
    ├─ Improves production query performance
    ├─ Subscriber is read-only (no conflicts)
    └─ Can subscribe multiple reporting servers
```

**Pattern 2: Cascading Replication for Data Warehouse**
```
Source OLTP System (Publisher A)
    │
    ├─ Subscriber B (Regional Distribution Center)
    │  │   (applies changes locally for regional analytics)
    │  │
    │  └─ Republisher of subset articles
    │       │
    │       └─ Subscriber C (Central Data Warehouse)
    │           │ (consolidates from multiple regions)
    │           │
    │           └─ Republisher for reporting layer
    │               │
    │               └─ Read-only Subscriber D (BI Database)
    │                   (Power BI, Tableau reports)

Use Case: Multi-region enterprise consolidating reporting
```

**Pattern 3: Merge Replication for Mobile/Branch Offices**
```
Central Headquarters (Publisher)      Branch Office (Subscriber)
├─ Master order data                  ├─ Can operate offline
├─ Customer master                    ├─ Download snapshot when connected
│                                     ├─ Make local updates
│ Merge Agent triggers                ├─ Conflicts resolved when merged
│ (daily or on-demand)                │
│                                     │
└─────────────────────────────────────┘

Workflow:
    1. Branch office disconnects, works offline
    2. Place orders locally, modify pricing
    3. Reconnect to network
    4. Merge Agent synchronizes
    5. Conflicts detected/resolved per policy

Supported by: VPN, intermittent WAN connections
```

#### DBA Best Practices for Replication Configuration

1. **Choose the Right Replication Type**
   - Snapshot: One-time or periodic full copies (data warehouse loading)
   - Transactional: Continuous, efficient change distribution (reporting, read-scale)
   - Merge: Bi-directional, conflict handling (mobile, disconnected scenarios)

2. **Monitor Distribution Agent Lag**
   - Query: `SELECT * FROM distribution.dbo.MSarticles`
   - Track: Last applied LSN vs. latest log LSN
   - Alert if lag exceeds SLA (e.g., SLA = 5 min lag, alert at 10 min)

3. **Use Push vs. Pull Subscription Strategically**
   - **Push Subscription**: Distributor initiates distribution; used for smaller subscriber networks
   - **Pull Subscription**: Subscriber initiates; used for large subscriber networks (reduces distributor load)

4. **Maintain Distributor Health**
   - Distributor disk must have sufficient space for pending commands
   - If distributor fills up, commands queue; replication stalls
   - Monitor: `SELECT * FROM distribution.dbo.MSrepl_commands`

5. **Implement Validation and Reconciliation**
   - Use `tablediff` utility to compare publisher vs. subscriber
   - Periodically validate subscription health
   - Re-initialize subscription if rows diverge

6. **Plan for Subscriber Failures**
   - Subscriber goes down; replication pauses until reconnected
   - When subscriber reconnects, it fetches missed commands from distributor
   - Ensure distributor retention period accommodates longest expected downtime

#### Common Pitfalls for Replication

| **Pitfall** | **Symptom** | **Root Cause** | **Fix** |
|---|---|---|---|
| **Distributor Disk Full** | Replication stalls; alerts fire | Too many pending commands; not cleaned up | Increase disk; tune retention policy; troubleshoot slow subscriber |
| **Lag Growing Unbounded** | Subscriber increasingly behind publisher | Network slow; subscriber CPU bottleneck; badly configured Distribution Agent | Increase Distribution Agent threads; analyze subscriber performance; check network |
| **Subscriber Out of Sync** | Row count mismatch between publisher and subscriber | Corruption; replication interrupted; invalid commands applied | Use `tablediff` to identify divergence; reinitialize subscription |
| **Replication Agent Job Failure** | Log Reader Agent or Distribution Agent job fails continuously | Syntax error in replicated commands; subscriber schema mismatch; permission issues | Check SQL Agent logs; test manually; add republish permissions |
| **Cascading Replication Breaks** | Intermediary publisher (Republisher) stops distributing to downstream | Intermediary database corrupted; replication interrupted; schema mismatch | Check Republisher health; re-initialize downstream subscription |
| **Merge Conflict Resolution Loop** | Same conflict re-occurs repeatedly | Custom conflict resolver has logic error; data continues to diverge | Review conflict resolver code; invalidate problematic rows on both sides |
| **Slow Snapshot Generation** | Snapshot takes hours for large table | Exclusive lock during snapshot blocks readers; network bandwidth to snapshot location limited | Use snapshot agent on subscriber; use incremental snapshots (SQL 2019+) |
| **Articles Not Replicating** | New rows on publisher don't appear on subscriber; old articles replicate fine | Article filtered incorrectly; filter join condition wrong; subscriber manually modified | Verify article filters; check subscriber for manual changes; reinitialize if needed |

---

## Log Shipping

### Textual Deep Dive

#### Internal Working Mechanism

Log Shipping is a lightweight, asynchronous disaster recovery mechanism that continuously backs up the transaction log from a Primary database, transfers the backup to Secondary (Standby) servers, and restores the logs there.

##### Log Shipping Architecture

```
PRIMARY DATABASE                    STANDALONE STANDBY
(Production, Read/Write)            (Warm Standby, Restore Pending)
│                                   │
├─ App Connections                  ├─ Monitors status (via query)
│  Read/Write Queries               │  Read-Only IF restored to recovery
│  (committed continuously)         │  (otherwise in "RESTORING" state)
│                                   │
└─ SQL Agent Backup Job             └─ SQL Agent Restore Job
   (Runs every N minutes: 15 min)      (Runs every N minutes: 5 min)
   │                                   │
   ├─ BACKUP LOG to \Backup\LSN123456  │
   │  (incremental log backup)         │
   │                                   │
   └─ Creates backup file              │
      "TransactionLog_20240328_0815.   │
       trn" (10 MB)                    │
      │                                │
      └─ File Share (UNC Path)         │  File Share Read Access
         \\BackupServer\LSBackups      │
          (High-speed network path)    │
         │                             │
         │ File Share Copy             │
         │ (by backup job or           │
         │  via Windows share          │
         │  replication)               │
         │                             │
         └────────────────────────────▶│
                                       │
                                       ├─ Restore Agent picks up file
                                       │
                                       ├─ RESTORE LOG with NORECOVERY
                                       │  (applies log to standby database)
                                       │
                                       └─ Updates hidden marker
                                          (track of last applied LSN)
```

##### Log Shipping Key Components

1. **Backup Job on Primary**
   - T-SQL: `BACKUP LOG [DatabaseName] TO DISK = '\\BackupShare\...\.trn'`
   - Runs every 5-15 minutes (configurable)
   - Generates incremental log backup
   - Backup file transferred to share (via file copy, DFS, or manual)

2. **Copy Job (Optional)**
   - Moves backup files from source share to destination share
   - Can be manual, or automated via Windows task scheduler
   - For distributed geographies (copy to multiple standby servers)

3. **Restore Job on Standby**
   - T-SQL: `RESTORE LOG [DatabaseName] FROM DISK = '...' WITH NORECOVERY`
   - Runs every 5-15 minutes (configurable; typically longer than backup interval)
   - Applies log blocks to standby database
   - Database remains in "RESTORING" state (not accessible for reading)
   - Can be made readable via snapshot or `RESTORE LOG ... WITH STANDBY` (read-only access)

4. **Monitor Job (Optional)**
   - Alerts if backup or restore jobs fail
   - Tracks RPO/RTO metrics

##### Log Shipping vs. Availability Groups vs. Replication

| **Aspect** | **Log Shipping** | **Availability Groups** | **Transactional Replication** |
|---|---|---|---|
| **Mechanism** | Log backup + restore | Real-time log streaming (sync/async) | Transaction extraction + apply |
| **RPO** | Log backup interval (5-60 min) | 0 (sync) to seconds (async) | 1-10 seconds |
| **RTO** | Manual; 10-60 minutes | Automatic; seconds | Manual; minutes-hours |
| **Failover** | Manual; no automatic promotion | Automatic (AG) or manual (forced) | Manual; requires script |
| **Complexity** | Simple (SQL jobs + file shares) | Moderate (cluster, quorum, WSFC) | Complex (agents, distribution DB) |
| **Cost** | Low (no licenses beyond SQL) | Moderate (depends on AG edition) | Moderate to higher |
| **Licensing** | Standard Edition OK | Standard has limits; Enterprise preferred | Standard Edition OK |
| **Scalability** | Single standby or multiple (daisy-chain) | Multiple replicas; cross-region possible | Many subscribers; cascading |
| **Read Workload** | Read-only standby (if recovered) | Read-only secondary replicas | Read-only subscribers |
| **Use Case** | Cost-effective DR, non-critical systems | Mission-critical HA/DR, auto-failover | Reporting, scaled read workload |

#### Production Usage Patterns

**Pattern 1: Single Standby for Cost-Effective DR**
```
Production Data Center              Disaster Recovery Site
├─ Primary Database                 ├─ Standby Database
│                                   │  (in RESTORING state,
│ SQL Agent Backup Job              │   not accessible until
│ (every 15 minutes)                │   recovered)
│ ├─ Backup transaction log         │
│ └─ Save to \\FileShare\Backups    │
│    │                              │
│    └─ Network Copy (via DFS)      │
│       │                           │
│       └─ Restore Job receives     │
│          ├─ Restore logs (every 15 min)
│          └─ Database stays in RESTORING

Standby Activation (Manual, if Primary fails):
    1. Stop log shipping restore job
    2. RESTORE LOG [...] WITH RECOVERY
       (Apply any remaining pending logs)
    3. Database becomes readable/writable
    4. Applications redirect to new primary

RPO: 15 minutes (lag time + unprocessed log)
RTO: 30-60 minutes (manual recovery + app redirection)
```

**Pattern 2: Multiple Geographically Distributed Standbys**
```
Primary Datacenter              Secondary DR Site 1         Secondary DR Site 2
├─ Production DB                ├─ Standby DB 1              ├─ Standby DB 2
│                               │                            │
└─ Backup Job (every 10 min)    │                            │
   ├─ Backup log                │                            │
   └─ \\AllRegions\Backups      │                            │
      │                         │                            │
      ├─────────────────────────┼────────────────────────────┤
      │                         │                            │
      │ Copy Job                │ Copy Job                   │
      │ to DR Site 1            │ to Site 2                  │
      ▼                         ▼                            ▼
   Restore Job 1         Restore Job 2         Restore Job 3

Activation Priority: Site 1 (closer) > Site 2 (farther)

Benefits:
    ├─ Geographic redundancy (survive entire regional failure)
    ├─ Cost-effective (no licenses for secondary servers)
    └─ Can serve reporting workload from one standby (recovered to read-only)
```

**Pattern 3: Cascading Log Shipping (Daisy-Chain)**
```
Primary (OLTP)
    │
    ├─ Backup Log (every 15 min)
    │
    └─ Share \\Primary\Backups
       │
       ├─ Primary Standby (Regional)
       │  ├─ Restore logs
       │  ├─ Also backs up logs (if Republishing enabled)
       │  │
       │  └─ Secondary Share \\Regional\Backups
       │     │
       │     └─ Secondary Standby (Remote DR)
       │        └─ Restores logs from Regional share

Use Case: Staged DR recovery
    1. If Primary fails, activate Regional standby (close, low latency)
    2. If entire region fails, activate Remote DR
    3. Simple, tiered RPO/RTO management
```

#### DBA Best Practices for Log Shipping Configuration

1. **Configure Appropriate Backup/Restore Intervals**
   - Backup interval should be 5-15 minutes (depends on RPO requirement)
   - Restore interval should be slightly longer (allow some lag, avoid contention)
   - Example: Backup every 15 min, Restore every 20 min

2. **Use File Shares on Fast, Dedicated Network**
   - File share should be on dedicated storage (not same LUN as production)
   - Network path should have redundancy (RAID, failover, DFS)
   - Verify file copy speed: minimum 10 MB/sec for typical log shipping

3. **Plan for Capacity and Retention**
   - Log backups accumulate; allocate disk accordingly
   - Retention: keep at least 1 week of log backups (troubleshooting needs)
   - Monthly cleanup script to archive/delete old logs

4. **Implement Robust Monitoring and Alerting**
   - Monitor: Last backup time vs. current time
   - Alert if backup fails or is delayed 30+ minutes
   - Track: Standby restore lag (how far behind primary)
   - Alert if restore lag exceeds SLA

5. **Test Failover Process Quarterly**
   - Practice promoting standby to primary (recovers database)
   - Measure actual RTO
   - Document failover steps; refine runbook

6. **Consider Read-Only Access (if business allows)**
   - If reporting workload exists, recover standby to read-only
   - Snapshot database for read-only access
   - Keep separate snapshot for reporting; update on schedule
   - Allows standbys to be productive during non-failure periods

#### Common Pitfalls for Log Shipping

| **Pitfall** | **Symptom** | **Root Cause** | **Fix** |
|---|---|---|---|
| **File Share Runs Out of Space** | Backup job fails; older backups deleted | Poor capacity planning; cleanup not running | Add more disk; implement retention policy; monitor disk usage |
| **Restore Lag Growing** | Standby falls behind; days of backups not restored | Network slow; restore job not running; standby server slow | Check network; verify job schedule; add standby resources |
| **Backup Files Don't Transfer** | Backup job runs but files don't appear on standby share | File share misconfigured; copy job not running; permission issues | Verify share permissions; check DFS replication; test copy job manually |
| **Failover Takes Too Long** | Promoted standby takes 2+ hours to recovery | Large volume of unapplied logs; recovery process slow | Track restore lag; apply logs more frequently; test failover time |
| **Lost Logs During Failover** | Data loss more than expected RPO | Primary crashed before backup completed; backup location lost | Use redundant storage; verify backups on multiple sites; minimize RPO |
| **Standby Unsynchronized After Restore** | Promoted standby has corrupted data | Log backup chain broken; network disruption; file corruption | Apply logs in correct order; re-initialize if needed; use BACKUP WITH VERIFY |
| **Automatic Failover Attempted (But Failed)** | Users confused; log shipping is manual only | Misunderstood difference between log shipping and AG | Clarify: log shipping requires manual promotion; consider AG if auto-failover needed |

---

### Practical Code Examples

#### Always On AG - Configuration

```sql
-- 1. CREATE AVAILABILITY GROUP (run on Primary)
CREATE AVAILABILITY GROUP AG_Production
    WITH (
        AUTOMATED_BACKUP_PREFERENCE = PRIMARY,
        FAILURE_CONDITION_LEVEL = 3,
        DB_FAILOVER = OFF,  -- SQL 2019+: Auto-failover at DB level
        DTC_SUPPORT = NONE,
        CLUSTER_TYPE = WSFC  -- Windows Server Failover Cluster
    )
    FOR DATABASE [MyProductionDB]
    REPLICA ON
        N'SQL-SERVER-01' WITH (
            ENDPOINT_URL = N'TCP://SQL-SERVER-01.contoso.com:5022',
            FAILOVER_MODE = AUTOMATIC,
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            BACKUP_PRIORITY = 100,
            PRIMARY_ROLE (ALLOW_CONNECTIONS = READ_WRITE)
        ),
        N'SQL-SERVER-02' WITH (
            ENDPOINT_URL = N'TCP://SQL-SERVER-02.contoso.com:5022',
            FAILOVER_MODE = AUTOMATIC,
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            BACKUP_PRIORITY = 50,
            SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY)
        ),
        N'SQL-SERVER-03' WITH (
            ENDPOINT_URL = N'TCP://SQL-SERVER-03.contoso.com:5022',
            FAILOVER_MODE = MANUAL,
            AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,
            BACKUP_PRIORITY = 0,
            SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY)
        );

-- 2. CREATE AG LISTENER
ALTER AVAILABILITY GROUP AG_Production
    ADD LISTENER N'AG_Listener_Production'
    (
        WITH IP (
            (N'10.0.1.100', N'255.255.255.0')
        ),
        PORT = 1433
    );

-- 3. CONNECTION STRING FOR APPLICATIONS
-- Server=AG_Listener_Production,1433;Database=MyProductionDB;...
-- Connection Timeout = 30 (retry if listener not responding)
-- Application Intent = ReadOnly (optional, for read-scale)

-- 4. VERIFY AG HEALTH
SELECT 
    ag.name AS AG_Name,
    ar.replica_server_name,
    ars.role_desc,
    drs.synchronization_state_desc,
    drs.is_commit_participant,
    drs.redo_queue_size  -- 0 = caught up
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id
JOIN sys.dm_hadr_database_replica_states drs ON ars.replica_id = drs.replica_id
WHERE ag.name = 'AG_Production'
ORDER BY ar.replica_server_name;

-- 5. MANUAL FAILOVER TO SECONDARY (if Primary is still accessible)
ALTER AVAILABILITY GROUP AG_Production FAILOVER;  -- Automatic failover

-- Force failover if Primary unreachable (data loss possible)
ALTER AVAILABILITY GROUP AG_Production 
FORCE_FAILOVER_ALLOW_DATA_LOSS;

-- 6. BACKUP STRATEGY WITH AG
-- Full backups can only be taken from Primary
BACKUP DATABASE [MyProductionDB]
TO DISK = N'\\BackupServer\BackupShare\MyProductionDB_FULL.bak'
WITH INIT, COMPRESSION, STATS = 10;

-- Log backups can be taken from any replica
BACKUP LOG [MyProductionDB]
TO DISK = N'\\BackupServer\BackupShare\MyProductionDB_LOG.trn'
WITH INIT, STATS = 10;

-- 7. READ-ONLY LISTENER FOR REPORTS
ALTER AVAILABILITY GROUP AG_Production
ADD LISTENER N'AG_Listener_ReadOnly'
(
    WITH IP (
        (N'10.0.1.101', N'255.255.255.0')
    ),
    PORT = 1434
);

-- Configure routing to secondary replicas (read-only)
ALTER AVAILABILITY REPLICA ON N'SQL-SERVER-02'
    WITH (
        SECONDARY_ROLE (
            ALLOW_CONNECTIONS = READ_ONLY,
            READ_ONLY_ROUTING_URL = N'TCP://SQL-SERVER-02.contoso.com:1433'
        )
    );

-- Connection string for reporting: Server=AG_Listener_ReadOnly,1434;...
-- ApplicationIntent = ReadOnly
```

#### Replication - Transactional Setup

```sql
-- =================================================================
-- TRANSACTIONAL REPLICATION SETUP (Step-by-Step)
-- =================================================================

-- PUBLISHER CONFIGURATION
-- ========================================

-- 1. Enable Publisher for Replication
EXEC sp_replicationdboption
    @dbname = N'MyProductionDB',
    @optname = N'publish',
    @value = N'true';

-- 2. Add Replication Publication
EXEC sp_addpublication
    @publication = N'PUB_ProductionData',
    @description = N'Publication of production data for reporting',
    @sync_method = N'concurrent',  -- allow concurrent snapshot
    @retention = 0,  -- indefinite retention
    @allow_push = N'true',
    @allow_pull = N'true',
    @immediate_sync = N'true';

-- 3. Add Articles to Publication (which tables to replicate)
EXEC sp_addarticle
    @publication = N'PUB_ProductionData',
    @article = N'Customers',
    @source_owner = N'dbo',
    @source_object = N'Customers',
    @type = N'logbased',
    @description = N'Customer master table',
    @destination_table = N'Customers',
    @creation_script = NULL,
    @pre_creation_cmd = N'drop';  -- drop/truncate if exists

EXEC sp_addarticle
    @publication = N'PUB_ProductionData',
    @article = N'Orders',
    @source_owner = N'dbo',
    @source_object = N'Orders',
    @type = N'logbased';

-- 4. Create Snapshot Agent Job
EXEC sp_addpublication_snapshot
    @publication = N'PUB_ProductionData',
    @frequency_type = 4,  -- Daily
    @frequency_interval = 1,
    @frequency_relative_interval = 0,
    @frequency_recurrence_factor = 0,
    @active_start_time_of_day = 20000,  -- 8 PM
    @active_start_date = 0,
    @active_end_time_of_day = 235959,
    @active_end_date = 0,
    @snapshot_job_name = NULL,
    @publisher_security_mode = 1,  -- Windows auth
    @job_login = NULL;

-- DISTRIBUTOR CONFIGURATION
-- ========================================

-- 1. Enable Distributor on separate instance (best practice)
EXEC sp_adddistributor
    @distributor = N'SQL-SERVER-DISTRIBUTOR',
    @password = N'YourComplexPassword123!';

-- 2. Add Distribution Database
EXEC sp_adddistributiondb
    @database = N'distribution',
    @data_folder = N'\\SQLServer\ReplicationData',
    @log_folder = N'\\SQLServer\ReplicationData',
    @min_distretention = 0,
    @max_distretention = 72;  -- 72 hours

-- 3. Add Publisher to Distributor
EXEC sp_adddistpublisher
    @publisher = N'SQL-SERVER-PRIMARY',
    @distribution_db = N'distribution',
    @security_mode = 1,  -- Windows auth
    @working_directory = N'\\SQLServer\ReplicationSnapshot';

-- SUBSCRIBER CONFIGURATION
-- ========================================

-- On Publisher (add pull subscription)
EXEC sp_addpullsubscription
    @publication = N'PUB_ProductionData',
    @subscriber = N'SQL-SERVER-REPORTING',
    @subscriber_db = N'ReportingDB',
    @subscription_type = N'pull',
    @sync_type = N'automatic',
    @article = N'all';

-- Define subscriber subscriber agent schedule
EXEC sp_addpullsubscription_agent
    @publication = N'PUB_ProductionData',
    @subscriber = N'SQL-SERVER-REPORTING',
    @subscriber_db = N'ReportingDB',
    @subscriber_security_mode = 1,
    @distributor_security_mode = 1,
    @frequency_type = 4,  -- Daily
    @frequency_interval = 1,
    @frequency_relative_interval = 0,
    @frequency_recurrence_factor = 1;

-- On Subscriber (add subscription)
USE ReportingDB;
EXEC sp_addsubscription
    @publication = N'PUB_ProductionData',
    @subscriber = N'SQL-SERVER-REPORTING',
    @destination_db = N'ReportingDB',
    @subscription_type = N'pull',
    @sync_type = N'automatic';

-- MONITORING & TROUBLESHOOTING
-- ========================================

-- View replication status
SELECT
    pub.name AS Publication,
    sub.name AS Subscriber,
    rep.status AS ReplicationStatus,
    rep.status_time AS LastStatusTime
FROM distribution.dbo.MSdistribution_agents rep
JOIN distribution.dbo.MSsubscriptions sub ON rep.subscription_id = sub.id
JOIN distribution.dbo.MSpublications pub ON rep.publisher_database_id = pub.id;

-- View replication commands pending on distributor
SELECT
    TOP 100
    mc.article_id,
    mc.xact_seqno,
    mc.command_time,
    mc.command_type,
    mc.command
FROM distribution.dbo.MSrepl_commands mc
ORDER BY mc.xact_seqno DESC;

-- Check subscriber for divergence
EXEC sp_table_validation
    @table = N'Customers',
    @expected_rowcount = 100,
    @rowcount_only = 0;
```

#### Log Shipping - Configuration

```sql
-- =================================================================
-- LOG SHIPPING SETUP (Manual Configuration)
-- =================================================================

-- PRIMARY INSTANCE - Backup Job
-- ========================================

-- Step 1: Create share for log backups
-- CMD (as Admin on backup server): mkdir \\BackupServer\LogShipping
-- ICACLS \\BackupServer\LogShipping /grant "Domain\SQL_Service:F"

-- Step 2: Create backup job on Primary
USE msdb;
EXEC sp_add_log_shipping_primary_database
    @database = N'MyProductionDB',
    @shiplog_frequency = 15,  -- Backup every 15 minutes
    @standby_server = N'SQL-SERVER-STANDBY',
    @standby_database = N'MyProductionDB',
    @backup_threshold = 45,  -- Alert if no backup in 45 min
    @threshold_alert_enabled = 1,
    @history_retention_period = 10080;  -- 7 days retention

-- Step 3: Add monitor server
EXEC sp_add_log_shipping_alert_job;

-- STANDBY INSTANCE - Restore Job
-- ========================================

-- Step 1: Create standby database (restore from FULL backup with NORECOVERY)
RESTORE DATABASE [MyProductionDB]
FROM DISK = N'\\BackupServer\Backups\MyProductionDB_FULL.bak'
WITH NORECOVERY, REPLACE;

-- Verify standby is in RESTORING state
SELECT database_id, name, state_desc FROM sys.databases WHERE name = 'MyProductionDB';
-- Expected: state = 1 (RESTORING)

-- Step 2: Add log shipping secondary database
EXEC sp_add_log_shipping_secondary_database
    @secondary_database = N'MyProductionDB',
    @primary_server = N'SQL-SERVER-PRIMARY',
    @primary_database = N'MyProductionDB',
    @restore_delay = 0,  -- No intentional delay (restore immediately)
    @restore_mode = 0,   -- Restore with NORECOVERY (standby not accessible)
    @disconnect_users = 1,
    @block_size = 64512,  -- Block size for restore
    @buffer_count = 15,
    @max_transfer_size = 1048576,
    @restore_threshold = 45,
    @threshold_alert_enabled = 1,
    @history_retention_period = 10080;

-- MONITORING & FAILOVER
-- ========================================

-- View log shipping status
SELECT * FROM msdb.dbo.log_shipping_primary_databases
WHERE primary_database = 'MyProductionDB';

SELECT * FROM msdb.dbo.log_shipping_secondary_databases
WHERE secondary_database = 'MyProductionDB';

-- View backup history
SELECT TOP 20
    h.database_name,
    h.backup_start_date,
    h.backup_finish_date,
    DATEDIFF(minute, h.backup_start_date, h.backup_finish_date) AS DurationMinutes
FROM msdb.dbo.backupset h
WHERE h.database_name = 'MyProductionDB'
AND h.type = 'L'  -- T=Full, D=Diff, L=Log
ORDER BY h.backup_start_date DESC;

-- Check restore lag (difference between primary and standby LSN)
SELECT
    primary_id,
    primary_server,
    primary_database,
    secondary_server,
    secondary_database,
    restore_threshold,
    last_restored_file,
    last_restored_date
FROM msdb.dbo.log_shipping_secondary_databases
WHERE secondary_database = 'MyProductionDB';

-- MANUAL FAILOVER (PROMOTE STANDBY TO PRIMARY)
-- ========================================

-- Step 1: Disable log shipping on primary (stop backup jobs)
EXEC sp_delete_log_shipping_primary_database
    @database = N'MyProductionDB';

-- Step 2: Apply final log (if available) on standby
RESTORE LOG [MyProductionDB]
FROM DISK = N'\\BackupServer\LogShipping\MyProductionDB_LATEST.trn'
WITH RECOVERY;  -- <-- WITH RECOVERY makes database fully readable/writable

-- Verify standby is online
SELECT database_id, name, state_desc FROM sys.databases WHERE name = 'MyProductionDB';
-- Expected: state = 0 (ONLINE)

-- Step 3: Delete log shipping secondary
EXEC sp_delete_log_shipping_secondary_database
    @secondary_database = N'MyProductionDB';

-- Step 4: Redirect applications to new primary
-- UPDATE CONNECTION STRINGS to point to SQL-SERVER-STANDBY

-- Step 5: Verify data integrity
DBCC CHECKDB (MyProductionDB, REPAIR_ALLOW_DATA_LOSS);
```

---

### ASCII Diagrams

#### Availability Group Failover Sequence

```
INITIAL STATE (Primary Healthy)

    APP LAYER
    ├─ Connection: Server=AG_Listener_Production;
    │  (Resolves to Primary: SQL-SERVER-01)
    │
    ▼
Primary (SQL-SERVER-01)          Secondary 1 (SQL-SERVER-02)
├─ Role: PRIMARY                 ├─ Role: SECONDARY
├─ State: SYNCHRONIZED           ├─ State: SYNCHRONIZED
├─ Synch: YES                    ├─ Synch: YES
└─ Accepting writes              └─ Read-only standby

        ↓ (Network/Hardware Failure on Primary)

FAILURE DETECTED

    Windows Failover Cluster
    ├─ Health check fails (3 misses)
    ├─ Primary deemed down
    └─ Initiates failover

Secondary 1 (SQL-SERVER-02)
├─ Elected as new Primary
├─ Completes any pending transactions
├─ Transitions to PRIMARY role
└─ Connections reopened

FIX COMPLETED

New Primary (SQL-SERVER-02)
├─ Role: PRIMARY
├─ State: SYNCHRONIZED (with new secondary)
└─ Accepting writes

APP LAYER
└─ DNS re-resolve: AG_Listener_Production
   (Now points to SQL-SERVER-02)
   Application reconnects (idle timeout triggers retry)
   Queries resume
```

#### Log Shipping Timeline

```
00:00                    06:00                   12:00                   18:00
  ├────────────────────────────┼────────────────────────────┼────────────────────────────┤
  │                            │                            │                            │
  Primary Database             Primary Database             Primary Database             Primary Database
  ├─ Transactions              ├─ Transactions              ├─ Transactions              ├─ Transactions
  └─ committed continuously    └─ committed continuously   └─ committed continuously   └─ committed continuously
                                      │                                    │
                                      │                                    │
  Backup Agent (every 15 min)        │ Backup runs                        │ Backup runs
  ├─ Backup Log                      │ ├─ MyProductionDB_202403281400.trn │ MyProductionDB_202403281600.trn
  └─ Save to share                   │ └─ Saved to \\BackupShare\         │
                                      │    ↓ File transfer (DFS)          │
                                      │    ├─ To Standby share            │
                                      │    └─ 2 seconds                   │
                                      │                                    │
                                      Standby Server                       △ Standby Server
                                      Restore Agent (every 15 min)        Restore Agent
                                      ├─ Detects new log backup           ├─ Applies pending logs
                                      ├─ Applies to standby DB            ├─ DB now 20 min behind
                                      └─ Standby now 15 min behind        └─ Ready for failover

RPO/RTO Analysis:
  ├─ RPO: 15 minutes (backup interval)
  │  └─ If primary fails at 06:07, standby missing 7 min of data
  │
  └─ RTO: 30-45 minutes (restore final logs + application failover)
     └─ Stop restore job, apply any pending logs, recover DB, 
        update connection strings, start applications
```

---

## Security Basics

### Textual Deep Dive

#### Internal Working Mechanism

SQL Server security operates on a **multi-layered principal-based model**. Every connection, query, and resource access is validated through:

1. **Authentication** – Verify identity (who are you?)
2. **Authorization** – Verify permissions (what can you do?)

##### Authentication Methods

```
┌────────────────────────────────────┐
│   Client Connection Request        │
│  (Username, Application, Host)     │
└────────────────┬────────────────────┘
                 │
         ┌───────▼───────┐
         │ Authentication│
         │    Method?    │
         └───┬───────┬───┘
             │       │
             ▼       ▼
       Windows   SQL Server
       Auth      Auth
         │           │
         │           ├─ Username + Password
         │           ├─ Sent over encrypted channel
         │           ├─ Validated against sys.sql_logins
         │           └─ ✓ (or ✗ Login failed)
         │
         ├─ Kerberos (domain)
         ├─ NTLM (local machine)
         ├─ Validate against Active Directory
         └─ ✓ (or ✗ Login failed)

         │
         ▼
    ┌─────────────────┐
    │   Connection    │
    │   Authenticated │
    └────────┬────────┘
             │
    ┌────────▼─────────┐
    │  Authorization   │
    │  (Server & DB    │
    │   roles)         │
    └─────────────────┘
```

**Windows Authentication** (Integrated Security)
- User's Windows identity used to authenticate to SQL Server
- No separate password required; uses domain credentials
- Kerberos protocol for secure token-passing
- **Advantage**: Centralized AD management, no password stored in SQL Server, audit trail via Windows Event Log
- **Disadvantage**: Requires Windows domain; more complex setup
- **Best Practice**: Use Windows Auth for internal applications; use SQL Auth only when Windows Auth impossible

**SQL Server Authentication**
- Username and password stored in SQL Server (sys.sql_logins)
- Password hashed using PBKDF2 or higher algorithms (SQL 2012+)
- Can optionally enforce password complexity policy
- **Advantage**: Works for non-domain scenarios (third-party apps, cloud scenarios without domain integration)
- **Disadvantage**: Passwords stored in DB; requires strong password policy and rotation; no audit trail integration with Windows
- **Best Practice**: Enforce strong passwords; rotate periodically; use only when Windows Auth not viable

##### Authorization Model (Server & Database Roles)

```
LOGIN (Authentication)
  │
  ├─ Authenticated identity recognized
  │  (Windows or SQL Server)
  │
  └─ USER (Authorization)
     └─ Mapped to database as database user
        │
        ├─ Can be assigned to Server Roles
        │  ├─ sysadmin (unrestricted access)
        │  ├─ serveradmin (server config)
        │  ├─ securityadmin (login/role management)
        │  ├─ processadmin (process control)
        │  ├─ ...others
        │  └─ dbcreator (database creation)
        │
        └─ Can be assigned to Database Roles
           ├─ db_owner (db admin)
           ├─ db_securityadmin (role management)
           ├─ db_ddladmin (schema changes)
           ├─ db_backupoperator (backup/restore)
           ├─ db_datareader (SELECT all)
           ├─ db_datawriter (INSERT/UPDATE/DELETE all)
           └─ Custom roles (self-defined, specific to business need)

PERMISSION RESOLUTION:
  ├─ Explicit GRANT (positive)
  ├─ Explicit DENY (absolute, overrides GRANT)
  └─ No grant = implicit reject (fail open/closed?)
```

**Server Roles** (Instance-level permissions):
- `sysadmin`: Full control; can do anything
- `serveradmin`: Configure server options and shutdown
- `securityadmin`: Manage logins and ALTER ANY LOGIN
- `processadmin`: Manage SQL Server processes
- `dbcreator`: Create/restore/drop databases
- `bulkadmin`: Can BULK INSERT
- `diskadmin`: Manage disk files
- `public`: Default; everyone has this

**Database Roles** (Database-level permissions):
- `db_owner`: Database owner; full control
- `db_securityadmin`: Manage users and permissions
- `db_ddladmin`: DDL (CREATE/ALTER/DROP)
- `db_backupoperator`: Backup/restore
- `db_datareader`: SELECT all
- `db_datawriter`: INSERT/UPDATE/DELETE all
- **Custom roles**: Create role with specific permissions

#### Production Usage Patterns

**Pattern 1: Windows/Active Directory for Internal Applications**
```
Active Directory
├─ Domain group: CONTOSO\SQLServerAdmins
├─ Domain group: CONTOSO\ReportingUsers
└─ Domain group: CONTOSO\AppServiceAccount

SQL Server
├─ LOGIN: [CONTOSO\SQLServerAdmins]
│  └─ Assigned to: sysadmin role (full admin)
│
├─ LOGIN: [CONTOSO\ReportingUsers]
│  └─ Assigned to: db_datareader role (read-only)
│     └─ Database user: "ReportingUsers"
│
└─ LOGIN: [CONTOSO\AppServiceAccount]
   └─ Assigned to: custom role "ApplicationRole"
      └─ Database user: "AppServiceUser"
         └─ Permissions: EXECUTE on stored procedures only

Benefit:
  ├─ Centralized identity management (AD)
  ├─ SSO for applications
  ├─ Audit trail in Windows Event Log
  └─ No passwords stored in SQL Server
```

**Pattern 2: SQL Server Auth for Third-Party Applications**
```
Third-Party Vendor Application
├─ Connects via: [ReportingApp]  (SQL auth login)
│  ├─ Password: '!Kx9#mP$vL2@nQ'
│  ├─ Stored: In app config (encrypted)
│  └─ Rotation: quarterly
│
└─ Assigned to: db_datareader (SELECT only)
   └─ Benefits: Least privilege; vendor app cannot modify data
```

**Pattern 3: Stored Procedure with EXECUTE AS Clause (Impersonation)**
```
Client connects as: [LowPrivilegeUser]
  ├─ Permissions: Only execute sp_ProcessOrders
  │
  └─ EXEC sp_ProcessOrders
     ├─ Stored procedure defined AS:
     │  CREATE PROCEDURE sp_ProcessOrders
     │  WITH EXECUTE AS OWNER
     │  BEGIN
     │      UPDATE dbo.Orders SET Status='Processed'
     │      ...
     │  END
     │
     └─ Executes under: [dbo] owner privileges
        └─ Allows low-privilege user to perform admin action
           safely & auditably

Benefits:
  ├─ User has minimal baseline permissions
  ├─ Sensitive operations encapsulated in stored procedures
  ├─ Auditable (sp_ProcessOrders logs who called it)
  └─ Cannot bypass controls (straightjacket)
```

#### DBA Best Practices for Security Configuration

1. **Principle of Least Privilege (POLP)**
   - Users should have ONLY the permissions needed to do their job
   - Avoid making everyone a db_owner or sysadmin
   - Review role assignments quarterly; remove unused access

2. **Use Windows Authentication Wherever Possible**
   - Easier to manage; centralized AD
   - Passwords not in SQL
   - Audit trail integration
   - Reserve SQL Auth for third-party applications only

3. **Strong Password Policy for SQL Logins**
   - Enforce `CHECK_POLICY = ON` (Windows complexity rules)
   - Minimum 14 characters, including uppercase, lowercase, numbers, symbols
   - Rotate every 90 days (especially for service accounts)
   - Never use built-in 'sa' for applications

4. **Disable SA Account (or Rename It)**
   - 'sa' is well-known; attackers target it first
   - Create separate admin account; disable 'sa' or rename it
   - If you must use 'sa', use strong password and disable when not needed

5. **Audit Access Regularly**
   - `SELECT * FROM sys.server_principals` (logins)
   - `SELECT * FROM sys.database_principals` (database users)
   - `SELECT * FROM sys.role_members` (role assignments)
   - Remove dormant accounts; clean up failed logins

6. **Default Database Isolation**
   - Each database gets default role: `public`
   - Explicit denies in `public` prevent implicit access
   - Example: `DENY SELECT ON dbo.SensitiveTable TO public`

#### Common Pitfalls for Security Basics

| **Pitfall** | **Consequence** | **Root Cause** | **Fix** |
|---|---|---|---|
| **Everyone is db_owner** | Accidental changes; destructive actions possible; no accountability | Convenience during setup; never cleaned up | Audit all role assignments; apply POLP; remove unnecessary db_owner |
| **SA Account Used for Everything** | Attacker compromises sa = full compromise | Default during setup; never changed | Rename/disable sa; create separate admin account with strong password |
| **Passwords in Config Files (Plain Text)** | If config file discovered, database compromised | Oversight; no encryption mechanism applied | Use Credential Manager or secure vaults (Azure Key Vault); encrypt connection strings |
| **Public Role Has Excessive Permissions** | Everyone can see sensitive data/execute functions | Default permissions never audited | Run discovery query; explicitly DENY sensitive objects to public |
| **SQL Auth Passwords Never Rotated** | Compromised password remains active indefinitely | No automated rotation process | Implement quarterly password change procedure; use service account credential rotation |
| **Logins Not Cleaned Up** | Dormant accounts become security liability; attack surface | Headcount decreased but logins never removed | Quarterly audit of sys.server_principals; remove unused logins |
| **\[BUILTIN\Administrators\] Granted Access** | All domain admins automatically get SQL access | Overly permissive authorization | Remove BUILTIN\Administrators; grant specific AD groups instead |
| **Stored Procedure EXECUTE AS SELF (Not as OWNER)** | User can escalate if they own procedure | Misunderstanding of EXECUTE AS semantics | Use EXECUTE AS OWNER (trusted); never SELF for privilege escalation |

---

## Advanced Security

### Textual Deep Dive

#### Internal Working Mechanism

Advanced security layers encrypt and control data at multiple levels: **at-rest (TDE), in-use (Always Encrypted), in-transit (TLS/SSL), and visibility (Auditing, RLS)**.

##### Transparent Data Encryption (TDE)

**Mechanism**: Database files (data + log) are encrypted using AES-256 at the storage layer. Encryption/decryption happens transparently at I/O time (no application code changes).

```
Application Query
  ├─ SELECT * FROM dbo.Customers
  ├─ (No idea data is encrypted)
  │
Buffer Pool (SQL Server Memory)
  └─ Data exists in CLEAR TEXT here
     (Encryption overhead only during disk I/O)

Disk I/O
  ├─ LazyWriter flushes dirty pages to disk
  │  ├─ Data Page (8 KB)
  │  └─ ① Encrypt page using Database Encryption Key (DEK)
  │     ├─ DEK encrypted using Certificate
  │     └─ Certificate encrypted using Service Master Key (SMK)
  │
  └─ Disk (physical storage)
     └─ Encrypted data (AES-256)

Read from Disk
  ├─ Read encrypted page from disk
  ├─ ① Decrypt using DEK (retrieved from SMK/Certificate chain)
  ├─ ② Bring into buffer pool (clear text)
  └─ ③ Query returns data (application sees clear text)

Key Hierarchy:
  Service Master Key (SMK) [STORED IN WINDOWS DATA PROTECTION API]
        │
        └─ Certificate (encrypted by SMK)
             │
             └─ Database Encryption Key (DEK) (encrypted by cert)
                  │
                  └─ Data files + Log files
```

**Properties:**
- **Scope**: Entire database (cannot encrypt specific tables/columns)
- **Performance**: Negligible overhead (< 5% in most workloads)
- **Backup**: Encrypted backups; restore on another server requires cert + SMK
- **Portability**: Backup encrypted with Cert A; restore to new server without Cert A = RESTORE fails

##### Always Encrypted

**Mechanism**: Data encrypted CLIENT-SIDE before sending to SQL Server. SQL Server stores encrypted data but cannot decrypt it (only client applications can).

```
Application Layer
├─ Column Encryption Key (CEK) [CLIENT-SIDE]
│  └─ Encrypted by Column Master Key (CMK) stored in Key Vault/Cert Store
│
Client Application
├─ SELECT CustomerNumber, SSN, Name
│  └─ WHERE SSN = '123-45-6789'
│
SQL Driver (ODBC/OLEDB)
├─ ① Intercepts query
├─ ② Encrypts SSN value using CEK
│  └─ '123-45-6789' → '0x... [256-bit ciphertext] ...'
├─ ③ Sends to SQL Server:
│    SELECT CustomerNumber, SSN, Name
│    WHERE SSN = 0x... [encrypted] ...
│
SQL Server (No Decryption Key)
├─ ① Receives encrypted query
├─ ② Searches for rows matching encrypted SSN value
│    (Deterministic encryption = same plaintext = same ciphertext)
├─ ③ Returns encrypted columns
│
SQL Driver
├─ ① Receives encrypted results
├─ ② Has CEK in memory (from application connection)
├─ ③ Decrypts SSN value
├─ ④ Returns clear text to application

Application
└─ Sees: CustomerNumber, SSN (decrypted), Name
```

**Key Points:**
- **Encrypted Columns Cannot Be**: Indexed (unless deterministic), sorted, GROUP BY'd, used in WHERE (unless deterministic)
- **Deterministic Encryption**: Same plaintext → Same ciphertext (enables equality searches but weaker)
- **Randomized Encryption**: Same plaintext → Different ciphertext each time (stronger, but no searches)
- **Performance**: Application-side encryption overhead; queries slightly slower

##### Auditing

**Mechanism**: SQL Server tracks all statements/actions; logs to table, file, or event log.

```
Connection Established
├─ Authentication event logged
│
Query Execution
├─ ① Audit Policy evaluated
│  ├─ Does this event match?
│  ├─ (e.g., "Log all DELETE statements")
│  └─ Action: LOG
│
├─ ② Audit record created
│  ├─ Timestamp
│  ├─ Principal (who executed)
│  ├─ Statement (what was executed)
│  ├─ Result (success/fail)
│  └─ Additional details (rows affected, etc.)
│
└─ ③ Audit destination
   ├─ sys.fn_get_audit_file() table
   ├─ Application Log (Windows Event Log)
   ├─ Or file (\\Audits\SQLServerAudit.sqlaudit)

Audit Query Example:
  SELECT event_time, server_principal_name, statement
  FROM sys.fn_get_audit_file('\\Audits\\SQLServerAudit*.sqlaudit', DEFAULT, DEFAULT)
  WHERE event_time > GETDATE() - 7  -- Last 7 days
  ORDER BY event_time DESC;
```

##### Row-Level Security (RLS)

**Mechanism**: Security predicates filter rows at query time; applied transparently to SELECT/UPDATE/DELETE operations.

```
User Query
├─ SELECT * FROM dbo.Orders
│  WHERE OrderAmount > 1000
│
Security Predicate Evaluation
├─ Add implicit WHERE condition
│  ├─ Based on USER_NAME() or CONTEXT_INFO()
│  ├─ Example: CustomerID = USER_NAME()
│  └─ Result: SELECT * FROM Orders
│          WHERE OrderAmount > 1000
│          AND CustomerID = @CurrentUser
│
Filtered Result
└─ Only rows for current user returned
   (even though query didn't explicitly filter)

Implementation:
  CREATE FUNCTION dbo.fn_SecurityPredicate(@CustomerID INT)
      RETURNS TABLE
      WITH SCHEMABINDING
  AS
  RETURN SELECT 1 AS result
      WHERE @CustomerID = CAST(SESSION_CONTEXT(N'CustomerID') AS INT)
      OR CAST(SESSION_CONTEXT(N'IsAdmin') AS INT) = 1;

  CREATE SECURITY POLICY dbo.CustomerSecurityPolicy
  ADD FILTER PREDICATE dbo.fn_SecurityPredicate(CustomerID)
  ON dbo.Orders
  WITH (STATE = ON);
```

#### Production Usage Patterns

**Pattern 1: TDE for Compliance (HIPAA/PCI-DSS)**
```
Requirement: Encrypt all patient/cardholder data at rest

Implementation:
  ├─ Enable TDE on all production databases
  ├─ Store certificates in secure backup location
  ├─ Monitor for key rotation events
  └─ Test restore process with encrypted backup

Benefit:
  ├─ Meets regulatory requirement for encryption
  ├─ Transparent (no application changes)
  ├─ Minimal performance impact
```

**Pattern 2: Always Encrypted for PII Columns**
```
Sensitive Columns: SSN, Medical Record Number, Credit Card

Implementation:
  ├─ Create Column Master Key (in Cert Store or Key Vault)
  ├─ Create Column Encryption Key (encrypted by CMK)
  ├─ Encrypt columns: SSN, MedicalRecordNumber, CreditCard
  │  (Determine: deterministic or randomized)
  ├─ Update applications to use encrypted connection
  └─ Test queries with sample data

Benefit:
  ├─ Even if database compromised, encrypted columns unreadable
  ├─ DBA cannot view PII in query results
  ├─ Meets regulations requiring column-level encryption
```

**Pattern 3: Auditing + Alerting for SOX Compliance**
```
Requirement: Track all CREATE/DROP/ALTER schema changes and DELETE operations

Implementation:
  CREATE SERVER AUDIT SPECIFICATION spec_audit_DDL_DML
  FOR SERVER AUDIT compliance_audit
  ADD (SUCCESSFUL_LOGIN_GROUP),
      (FAILED_LOGIN_GROUP),
      (SCHEMA_OBJECT_CHANGE_GROUP),
      (DATABASE_OBJECT_CHANGE_GROUP)
  WITH (STATE = ON);

  └─ Alert if unauthorized DDL detected
     ├─ Automated email to security team
     └─ Escalate for investigation

Benefit:
  ├─ Regulatory audit trail
  ├─ Detect unauthorized schema changes
  ├─ Non-repudiation (proof of who did what)
```

**Pattern 4: Row-Level Security for SaaS (Multi-Tenant)**
```
Scenario: SaaS application serving 1000 customers; single database

Requirement: Customer A sees only Customer A's data; isolation enforced

Implementation:
  ├─ TABLE: dbo.Customers (CustomerID, Name, ...)
  ├─ TABLE: dbo.Orders (OrderID, CustomerID, Amount, ...)
  │
  ├─ Add security predicate:
  │  WHERE CustomerID = (SELECT context_info() AS CustomerID)
  │
  └─ When Customer A connects:
     ├─ Set session context @CustomerID = 5
     ├─ SELECT * FROM Orders returns ONLY OrderID, Amount where CustomerID=5
     └─ Even if query doesn't filter by CustomerID

Benefit:
  ├─ True data isolation without separate databases
  ├─ Cost-effective multi-tenancy
  ├─ Prevents cross-customer data leakage from SQL injection
```

#### DBA Best Practices for Advanced Security

1. **Implement Defense-in-Depth**
   - Don't rely on single encryption layer
   - Combine: TDE (at-rest) + Always Encrypted (in-use) + TLS (in-transit)
   - Add audit layer for visibility

2. **TDE Master Key Strategy**
   - Store Service Master Key (SMK) backup in secure vault
   - Certificate backup in separate secure location
   - Test restore procedure annually
   - Monitor certificate expiration

3. **Always Encrypted Key Management**
   - Store Column Master Keys in Azure Key Vault (not on server)
   - Rotation of Column Encryption Keys annually
   - Access control: Only application service accounts have keystore access

4. **Audit Configuration Hygiene**
   - Start audit disabled; enable when needed
   - Specify clear retention policy (90 days typical)
   - Alert on audit failure (if audit fails, app should fail safe)
   - Review audit logs weekly for anomalies

5. **Row-Level Security Testing**
   - Test with users from different security contexts
   - Verify predicate applied at all access methods (direct query, app, reporting tool)
   - Monitor performance impact (predicates add WHERE clause overhead)

#### Common Pitfalls for Advanced Security

| **Pitfall** | **Consequence** | **Root Cause** | **Fix** |
|---|---|---|---|
| **TDE Enabled, Certificate Lost** | Database unrecoverable; cannot restore | No backup of certificate; SMK key rotated without export | Export certificate after creation; store in secure vault; document process |
| **Always Encrypted Columns Indexed** | Queries fail; performance issue | Developer forgot Always Encrypted limitation | Apply Always Encrypted after index creation; use non-encrypted columns for filtering |
| **Audit File Fills Up Disk** | Audit stops; application stops writing data | No retention policy; audit not cleaned up | Implement retention policy (90 days); archive old audit files; monitor disk space |
| **Audit Predicate Too Broad** | Massive overhead; all queries slower | Audit entire database; not scoped to risky operations | Scope audit to specific tables/operations (SCHEMA_OBJECT_CHANGE, DELETE only) |
| **RLS Predicate Logic Error** | User sees data they shouldn't (breach!) | Predicate condition incorrect; SESSION_CONTEXT not set | Test with multiple user contexts; include negative test cases; verify predicate logic |
| **Always Encrypted CMK Compromised** | All encrypted data compromised | Key stored insecurely (plain-text file, unencrypted backup) | Store in Key Vault with RBAC access; rotate key immediately; re-encrypt data |
| **Certificate Expires During Production Use** | TDE stops working; queries fail with "Certificate Expired" | No monitoring; expiration date not tracked | Set reminder 6 months before expiration; test renewal procedure early |

---

## Maintenance

### Textual Deep Dive

#### Internal Working Mechanism

Database Maintenance encompasses routine tasks that preserve performance, integrity, and recoverability: **index fragmentation reduction, statistics updates, integrity checks, backup management**.

##### Index Fragmentation & Maintenance

**Root Cause of Fragmentation:**
```
Index B-Tree Structure (CLUSTERED INDEX on PRIMARY KEY)

Initial State (Efficient)
─────────────────────────
Leaf Level (Data Pages):
├─ Page 1: PK=1-100   (100% full, sequential)
├─ Page 2: PK=101-200 (100% full, sequential)
└─ Page 3: PK=201-300 (100% full, sequential)

Query Performance:
  └─ SELECT * WHERE PK=150
     ├─ Binary search: Find Page 2
     └─ Sequential scan within page

After Deletes/Inserts (Fragmented)
──────────────────────────────────
├─ Page 1: PK=1, 50, 150, 300      (25% full, scattered)
├─ Page 2: PK=101, 220, 280         (30% full, scattered)
├─ Page 3: PK=201, 350, 410         (30% full, scattered)
└─ Page 4: PK=401-500               (100% full)

Query Performance:
  └─ SELECT * WHERE PK 200-300
     ├─ Initially at Page 3
     ├─ Next match at Page 1 (seek + pause)
     ├─ Next match at Page 2 (seek + pause)
     └─ Many disk seeks = POOR PERFORMANCE

Fragmentation Metrics:
  ├─ Logical Fragmentation: % of wrong-order pages
  │  └─ < 10%: REORGANIZE, > 10%: REBUILD
  │
  └─ Avg Fragmentation %:
       (100 - (internal fragmentation% + external fragmentation%))
```

**Index Rebuild vs. Reorganize:**

```
REORGANIZE (Online, No Lock)
├─ Compact existing pages
├─ Remove free space
├─ Low impact; slow
├─ Cost: ~2-5% CPU, 1-2 hours (1 TB index)
├─ Can run during business hours (but slower)
└─ Good for: 10-20% fragmentation

REBUILD (Offline Default, Can be Online)
├─ Drop and recreate index (full rewrite)
├─ Highly efficient; fast
├─ Can lock table (non-ONLINE rebuild)
├─ Cost: ~50% CPU spike, 5-10 minutes (1 TB index)
├─ Must run during maintenance window
└─ Good for: > 20% fragmentation OR index corruption

Decision Matrix:
  Fragmentation < 10%: No action needed
  Fragmentation 10-20%: REORGANIZE (online)
  Fragmentation > 20%: REBUILD (online if available)
```

##### Statistics Maintenance

**Root Cause of Stale Statistics:**
```
Initial State (Fresh Statistics)
──────────────────────────────
Table: Customers (1,000 rows)
  └─ Statistics: Created at 1,000 rows
     └─ Query Optimizer makes plan for ~1,000 rows

After 9 Months (Data Grows)
────────────────────────────
Table: Customers (10,000,000 rows)
  └─ Statistics: Still say 1,000 rows (STALE!)
     
Query: SELECT * FROM Customers WHERE Region='East'
     ├─ Optimizer thinks: ~10 rows (1% of 1,000)
     ├─ Reality: ~1,000,000 rows (10% of 10,000,000)
     ├─ Plan chosen: Seeks (fast for small result sets)
     └─ Actual result: Table scan needed (slow for 1M rows)
        └─ QUERY PERFORMANCE DEGRADED!

Statistics Update Trigger:
├─ Auto-Update: After ~20% of rows change
├─ Manual-Update: UPDATE STATISTICS (force refresh)
└─ Async Update: Stats updated in background (no blocking)
```

##### Integrity Checks (DBCC CHECKDB)

**Root Cause of Corruption:**
```
Disk Bit-Flip (by cosmic ray, bad sector, etc.)
  └─ Page stored as: 0xABCD1234... CORRUPTED!

Corrupt Page Read Into Memory
  ├─ SQL Server cache: [CORRUPTED DATA]
  ├─ Query uses corrupt data
  │  └─ Wrong customer orders retrieved
  │  └─ Reports show $$ in wrong places
  └─ Corruption replicates to secondaries (all affected)

DBCC CHECKDB Scan
  ├─ Read every page sequentially
  ├─ Validate page structure (headers, links)
  ├─ Validate logical consistency (FK relationships)
  ├─ Detect: Page checksums don't match
  ├─ Report: "Error 8645: Logical consistency error"
  └─ Action: REPAIR_REBUILD or business decision (restore from backup)
```

#### Production Usage Patterns

**Pattern 1: Proactive Maintenance Window (Weekly)**
```
Saturday 2 AM - 5 AM (Low traffic window)

Step 1: Index Fragmentation Analysis
  ├─ Query DMV: sys.dm_db_index_physical_stats
  ├─ Identify: Fragmentation > 10%
  └─ Action: REORGANIZE or REBUILD

Step 2: Statistics Update
  ├─ Query: sys.objects s WHERE auto_update_statistic=0
  ├─ Execute: UPDATE STATISTICS (all tables)
  └─ Time: Depends on # tables; 30-120 min

Step 3: Integrity Check
  ├─ Execute: DBCC CHECKDB
  ├─ If corrupted: Restore from backup
  └─ If OK: Log "No corruption detected"

Step 4: Index Cleanup
  ├─ Remove fragmented unused indexes
  ├─ Query: sys.dm_db_index_usage_stats (no activity)
  └─ Drop: Unused indexes (save disk, improve INSERT/UPDATE)

Estimated Duration: 1-3 hours (DB size dependent)
Impact: During maintenance window only; acceptable
```

**Pattern 2: Real-Time Monitoring + On-Demand Maintenance**
```
Monitoring (Continuous)
  ├─ Query performance baseline established
  ├─ Alert if query runtime > 110% of baseline
  └─ Trigger: "Index Fragmentation Check"

Detected: SELECT query suddenly slow (50 sec vs 2 sec)
  ├─ Diagnosis: Fragmentation > 30%
  ├─ Cause: Bulk INSERT yesterday fragmented index
  └─ Action: Immediate REBUILD (business requires)
     ├─ Online REBUILD with WAIT_AT_LOW_PRIORITY
     ├─ Duration: 10 minutes
     └─ Result: Query back to 2 seconds

Benefit:
  ├─ Reactive approach (fix when needed)
  └─ No pre-scheduled maintenance blocking
```

**Pattern 3: Distributed Maintenance (HA/DR Scenario)**
```
Studio Setup: Availability Group with 2 secondaries

Challenge: Cannot REBUILD index on primary during business hours
Solution: Distribute maintenance to secondary replicas

Process:
  ├─ Day 1: Disable replication; REBUILD on Secondary 1
  │         Re-enable replication (catchup)
  │
  ├─ Day 2: Disable replication; REBUILD on Secondary 2
  │         Re-enable replication (catchup)
  │
  └─ Day 3: Failover to Secondary 1 (becomes new primary)
            └─ Primary is now fully-rebuilt
               (REBUILD already done during standby phase)

Benefit:
  ├─ Maintenance window avoided
  ├─ Zero downtime
  └─ Both replicas properly maintained
```

#### DBA Best Practices for Maintenance

1. **Index Maintenance Strategy**
   - Assess fragmentation weekly (or nightly)
   - Fragmentaton 10-20% → REORGANIZE (online, during business hours)
   - Fragmentation > 20% → REBUILD (scheduled maintenance window)
   - Use `INDEX REBUILD ... WITH (FILLFACTOR = 80)` to reserve space

2. **Statistics Maintenance**
   - Auto-update enabled by default (usually sufficient)
   - For data warehouse: Schedule `UPDATE STATISTICS` nightly (after bulk load)
   - Update full statistics (not just incremental) monthly

3. **DBCC CHECKDB Strategy**
   - Run weekly minimum (full database integrity check)
   - Use `CHECKDB WITH NO_INFOMSGS` to suppress routine messages
   - Create alert if corruption detected
   - Have backup restore procedure ready

4. **Automate Maintenance Using SQL Agent**
   - Create maintenance jobs (index rebuild, stats update, checkdb)
   - Schedule during low-traffic maintenance window (2-5 AM Saturdays)
   - Implement retry logic for failed jobs
   - Monitor job execution; alert on failure

5. **Distributed Maintenance for HA/DR**
   - Run REBUILD on secondary to avoid primary interruption
   - Disable availability group SEEDING during heavy maintenance
   - Validate replica sync after maintenance

#### Common Pitfalls for Maintenance

| **Pitfall** | **Consequence** | **Root Cause** | **Fix** |
|---|---|---|---|
| **No Maintenance Schedule** | Fragmentation accumulates; performance degrades over months | "We'll do it when needed" mentality | Implement weekly schedule; automate via SQL Agent |
| **REBUILD During Business Hours** | Locks table; users blocked; complaints | Didn't plan maintenance window | Schedule during off-hours; use ONLINE REBUILD (SQL 2012+) |
| **Statistics Not Updated After Bulk Insert** | Query optimizer uses stale stats; bad plans generated | Forgot to run UPDATE STATISTICS | Document: After bulk insert, UPDATE STATISTICS; automate if possible |
| **CHECKDB Never Run** | Corruption exists undetected for months | "No time for maintenance" | Schedule nightly CHECKDB; alert on error |
| **Full Table Scans Due to High Fragmentation** | Queries run 10x slower; CPU spikes | Fragmentation never addressed | Implement fragmentation monitoring; REBUILD when > 20% |
| **Index Bloat (Missing Indexes + Unused Indexes)** | Larger backup sizes; slower restore; wasted disk space | Indexes never audited; cleanup job not run | Audit index usage monthly; drop unused indexes |
| **Recovery Model Not Set Correctly** | Backups not created; can't do point-in-time recovery | Set to SIMPLE for production; confusion about recovery models | Set production to FULL; perform log backups every 15 minutes |

---

## Monitoring & Troubleshooting

### Textual Deep Dive

#### Internal Working Mechanism

Effective monitoring captures performance metrics, identifies bottlenecks, and enables proactive remediation. **DMVs** (Dynamic Management Views) provide real-time system state; **Extended Events** capture historical diagnostic data.

##### Dynamic Management Views (DMVs) – Real-Time Diagnostics

```
DMV Categories:
│
├─ Server State DMVs
│  ├─ sys.dm_os_tasks (active tasks)
│  ├─ sys.dm_os_waiting_tasks (blocked/waiting)
│  ├─ sys.dm_os_memory_clerks (memory consumption)
│  └─ sys.dm_os_sys_info (CPU count, instance memory)
│
├─ Execution DMVs
│  ├─ sys.dm_exec_requests (active queries)
│  ├─ sys.dm_exec_query_stats (historical query stats)
│  ├─ sys.dm_exec_sql_text (query text from plan_handle)
│  └─ sys.dm_exec_query_plan (execution plan)
│
├─ Waiting/Blocking DMVs
│  ├─ sys.dm_tran_locks (active locks)
│  ├─ sys.dm_exec_blocking_requests (who blocks whom)
│  ├─ sys.dm_os_wait_stats (cumulative wait times)
│  └─ sys.dm_tran_active_transactions (open transactions)
│
├─ Performance DMVs
│  ├─ sys.dm_io_virtual_file_stats (disk I/O stats)
│  ├─ sys.dm_db_index_usage_stats (index usage)
│  └─ sys.dm_exec_procedure_stats (proc execution stats)
│
└─ Resource Governor DMVs
   ├─ sys.dm_resource_governor_workload_groups
   └─ sys.dm_resource_governor_resource_pools
```

**DMV Query Workflow:**

```
Baseline Established (Week 1)
├─ CPU: 20%
├─ Memory: 60%
├─ Disk I/O: 100 IOPS
└─ Query Runtime: 2 sec (SELECT * FROM Orders)

Monitor Continuously (DMVs Queried Every 5 Min)
├─ Week 2: CPU 22% (normal variation)
├─ Week 3: CPU 18% (normal)
├─ Week 4: CPU 65% (!!)
│      └─ ALERT: 3x baseline
│         └─ Investigate DMVs:
│            ├─ sys.dm_exec_requests (active queries)
│            ├─ sys.dm_os_waiting_tasks (blocked?)
│            ├─ sys.dm_tran_locks (deadlocks?)
│            └─ sys.dm_exec_query_stats (query regressions?)

Investigation Results
├─ Found slow query: SELECT * FROM Orders (now 15 sec!)
├─ Root cause: JOIN to missing index, table scan
└─ Fix: Create index (0.5 sec after fix)
   └─ CPU back to 20%
```

**Key DMV Queries for Troubleshooting:**

```
1. WHO IS BLOCKING WHOM?
   └─ sys.dm_exec_blocking_requests
      └─ Session 45 waiting on Session 32
      └─ Clear action: KILL SESSION 32 (if safe)

2. WHAT QUERY IS RUNNING NOW?
   └─ sys.dm_exec_requests (active session)
      └─ Execution plan from dm_exec_query_plan
      └─ Identify: Is table scan? Missing index?

3. HAS PERFORMANCE DEGRADED?
   └─ sys.dm_exec_query_stats
      └─ Compare: Execution count, average duration vs baseline
      └─ Identify: Regression after schema change?

4. WHAT'S CONSUMING RESOURCES?
   └─ sys.dm_os_memory_clerks (memory)
      sys.dm_io_virtual_file_stats (disk I/O)
      sys.dm_exec_requests (CPU)
```

##### Extended Events (Historical Tracing)

**Purpose**: Capture detailed diagnostic data (queries, deadlocks, waits) persisted to disk/live tracker.

```
Extended Event Session (Persistent Data Capture)

Configuration:
├─ Event: "sql_statement_completed" (every statement)
├─ Filter: duration > 1000 ms (> 1 second)
├─ Targets:
│  ├─ event_file (disk file; survives instance restart)
│  └─ ring_buffer (in-memory; lost on restart)
│
└─ Max file size: 50 MB (rotate files)

Captured Data (Example)
├─ Timestamp: 2024-03-28 14:32:15.123
├─ Event: sql_statement_completed
├─ Duration: 5000 ms (5 sec query)
├─ CPU Time: 3000 ms
├─ Logical Reads: 50000 (table scans)
├─ Logical Writes: 100
├─ Rows Returned: 10
├─ Statement: SELECT * FROM Orders WHERE Region='West'
├─ Database: ProductionDB
├─ Session: 45
├─ Principal: Domain\ReportingUser
└─ Client: 192.168.1.100

Query Extended Events File
├─ Aggregate: All queries > 1 sec from past 7 days
├─ Sort by: Duration DESC
├─ Find: Slowest queries, trends, patterns
└─ Action: Create index, add partition, rewrite query
```

##### Wait Statistics Analysis

**Concept**: SQL Server accumulates cumulative wait time by wait type; high waits indicate bottleneck.

```
Wait Type Analysis:

Total Wait Time Breakdown (Example Database)
├─ THREADPOOL (33%) – Not enough threads; work queued
├─ PAGEIOLATCH (25%) – Slow disk I/O; waiting for page
├─ LCK_M (15%) – Locks; blocked by other queries
├─ LATCH (12%) – Memory structure contention
├─ CPU_QUEUE (10%) – CPU oversubscribed
├─ TEMPDB (5%) – Congestion in temp database
└─ Other (5%)

Bottleneck Identification:
├─ PAGEIOLATCH dominant? → Issue: Disk I/O
│  └─ Action: Add more disk, check backups/fragmentation
│
├─ LCK_M dominant? → Issue: Blocking
│  └─ Action: Review lock escalation, query design
│
├─ THREADPOOL dominant? → Issue: CPU or parallel query congestion
│  └─ Action: Increase worker threads, reduce query parallelism
│
└─ TEMPDB dominant? → Issue: Tempdb contention
   └─ Action: Add more tempdb files, tune queries to reduce temp usage
```

#### Production Usage Patterns

**Pattern 1: Baseline Performance Monitoring**
```
Week 1 (Establish Baseline)
├─ CPU: 25%, Memory: 60%, Disk: 80 IOPS
├─ Top 10 Queries (by duration)
├─ Query 1: SELECT (5 sec) [expected]
├─ Query 2: Report (3 sec) [expected]
└─ Wait stats: PAGEIOLATCH (40%), LCK_M (30%)

Week 2-4 (Compare Against Baseline)
├─ CPU: 22%, Memory: 58% → Normal variation
├─ Query 1: 5 sec (same) ✓
├─ Query Report: 15 sec ✗✗✗ REGRESSION!
└─ Wait stats: PAGEIOLATCH (20%), LCK_M (50%) ← Locks increased!

Issue Diagnosed
├─ Report query suddenly blocking other sessions
├─ Root cause: Missing index added to WHERE clause
└─ Fix: Create index (Query back to 3 sec)
```

**Pattern 2: Blocking/Deadlock Investigation**
```
Application Error: "Deadlock detected; transaction rolled back"

Investigation Steps:
├─ Step 1: Review Extended Event for deadlock graph
│  ├─ Session 12: UPDATE Orders WHERE OrderID=5
│  ├─ Session 15: UPDATE OrderDetails WHERE OrderID=5
│  └─ Circular dependency: Both wait on each other
│
├─ Step 2: Check for blocking
│  └─ sys.dm_exec_blocking_requests
│     └─ Session 12 blocked by Session 15 (waiting 30 sec)
│
├─ Step 3: Identify query
│  └─ Both sessions executing same report procedure
│
└─ Step 4: Root cause analysis
   ├─ Application issues batch of updates
   ├─ Two sessions process same order ID
   └─ Lock escalation: Row lock → Page lock → Table lock
      └─ Deadlock occurs

Solution:
├─ Add NOLOCK hint (report reads)
├─ Reduce transaction scope (update fewer rows per transaction)
└─ Implement retry logic in application (re-execute if deadlock)
```

**Pattern 3: Continuous Alerting & Remediation**
```
Monitoring Agent (Run Every minute)
├─ Query: sys.dm_exec_requests + sys.dm_os_wait_stats
├─ Check: CPU > 80%, Memory > 90%, Wait > baseline + 50%
└─ Action: IF condition THEN alert

Alert Threshold Triggered
├─ CPU spiked to 95% (was 25%)
├─ Top query: SELECT * FROM Customers (table scan, slow)
├─ Diagnosis: Missing index on WHERE clause
└─ Automatic action (if script approved):
   ├─ Launch Extended Event trace (1-hour session)
   ├─ Send email to DBA: "CPU spike detected; investigating"
   ├─ Optionally: KILL long-running query (if rule allows)
   └─ Recheck metrics (if CPU drops, resolved; if not, escalate)

Remediation
├─ DBA reviews Extended Event
├─ Creates index (or rewrites query)
├─ Deploys fix in next maintenance window
└─ Monitors improvement metrics
```

#### DBA Best Practices for Monitoring

1. **Establish Baseline Metrics First**
   - CPU, Memory, Disk I/O, Query Duration for well-known queries
   - Wait stats aggregated (PAGEIOLATCH normally high for OLTP)
   - Expected query runtimes per business requirement

2. **Alert on Anomalies, Not Absolute Values**
   - CPU > 80% is normal in peak times; alert if > 90% AND unusual
   - Query duration: Alert if > 150% of baseline, not > 30 seconds (baseline varies)

3. **Use Extended Events for Deep Context**
   - DMVs lightweight; safe to query continuously
   - Extended Events provide details (wait types, I/O, full query text)
   - Combine both: DMVs detect issue, Extended Events explain why

4. **Monitor HA/DR Health Specifically**
   - AG: Synchronization state, listener health
   - Replication: Lag (subscriber LSN vs publisher LSN)
   - Log Shipping: Last backup time, restore lag

5. **Implement Retention Policy for Monitoring Data**
   - Extended Event files grow (rotate hourly/daily)
   - Archive to central repository (long-term analysis)
   - Purge old data per retention policy (6 months typical)

#### Common Pitfalls for Monitoring

| **Pitfall** | **Consequence** | **Root Cause** | **Fix** |
|---|---|---|---|
| **Alerts Not Tuned** | Alert fatigue; admins ignore real alerts | Alert thresholds too low; too many false alarms | Establish baseline; tune thresholds (90%ile performance) |
| **DMV Queries Impact Performance** | Querying DMVs adds load; makes problem worse | Overly complex joins; running too frequently | Simple DMV queries; run every 5+ minutes, not every second |
| **Extended Events Not Configured** | No historical data to debug issues | "Can investigate later" mentality | Configure permanent Extended Event session (event_file) |
| **Monitoring Data Fills Up Disk** | Extended Event files grow unbounded; system runs out of space | No retention policy; cleanup job not run | Implement automatic file rotation; archive old files |
| **Blocking Not Investigated** | Silent performance degradation; users complain | No blocking monitor in place | Query sys.dm_exec_blocking_requests every 10 minutes; alert |
| **HA/DR Health Blindspot** | Failover fails because replica is lagging | Only monitoring primary; secondary health ignored | Monitor AG sync state, replication lag, listener health |
| **Query Regression Undetected** | Performance issue attributed to "random slowness" | No query baseline; performance degradation gradual | Establish query runtime baseline; trend analysis (% change over time) |

---

### Practical Code Examples

#### Monitoring Query Examples

```sql
-- =================================================================
-- ACTIVE QUERY MONITORING
-- =================================================================

-- 1. WHO IS CURRENTLY RUNNING QUERIES?
SELECT
    session_id,
    login_name,
    host_name,
    database_id,
    command,
    status,
    cpu_time,
    total_elapsed_time,
    reads,
    writes,
    logical_reads,
    text,
    statement_start_offset,
    statement_end_offset
FROM sys.dm_exec_requests req
CROSS APPLY sys.dm_exec_sql_text(sql_handle) txt
WHERE session_id > 50  -- Exclude system sessions
ORDER BY cpu_time DESC;

-- 2. QUERY BLOCKING ANALYSIS
SELECT
    blocking.session_id AS BlockedSessionID,
    blocking.login_name AS BlockedUser,
    blocked_query.text AS BlockedQuery,
    blocking_request.session_id AS BlockingSessionID,
    blocking_request.login_name AS BlockingUser,
    blocking_query.text AS BlockingQuery,
    DATEDIFF(second, blocking_request.start_time, GETDATE()) AS BlockedDurationSeconds
FROM sys.dm_exec_requests blocking_request
INNER JOIN sys.dm_exec_requests blocking 
    ON blocking_request.session_id = blocking.blocking_session_id
CROSS APPLY sys.dm_exec_sql_text(blocking.sql_handle) blocked_query
CROSS APPLY sys.dm_exec_sql_text(blocking_request.sql_handle) blocking_query
ORDER BY BlockedDurationSeconds DESC;

-- 3. WAIT STATISTICS (CUMULATIVE SINCE STARTUP)
SELECT TOP 20
    wait_type,
    waiting_tasks_count AS WaitCount,
    wait_time_ms / 1000 AS WaitTimeSeconds,
    signal_wait_time_ms / 1000 AS SignalWaitTimeSeconds,
    CAST(100.0 * wait_time_ms / SUM(wait_time_ms) OVER () AS NUMERIC(5, 2)) AS PercentageOfWaits
FROM sys.dm_os_wait_stats
WHERE wait_type NOT IN (
    'CLR_SEMAPHORE', 'DBMIRROR_EVENTS_QUEUE', 'LAZYWRITER_SLEEP',
    'XE_TIMER_EVENT', 'XE_DISPATCHER_JOIN', 'QDS_CLEANUP_STALE_QUERIES',
    'ONDEMAND_TASK_QUEUE', 'REQUEST_FOR_DEADLOCK_SEARCH', 'LOGMGR_QUEUE',
    'CHECKPOINT_QUEUE', 'FUTEX_QUEUE'
)
ORDER BY wait_time_ms DESC;

-- 4. SLOW QUERIES (QUERY STATS)
SELECT TOP 20
    CAST(CAST(qs.total_elapsed_time / 1000000 AS NUMERIC(10, 2)) AS FLOAT) AS TotalElapsedTime_sec,
    qs.execution_count,
    CAST(CAST(qs.total_elapsed_time / 1000000 / NULLIF(qs.execution_count, 0) AS NUMERIC(10, 2)) AS FLOAT) AS AvgElapsedTime_sec,
    qs.last_execution_time,
    SUBSTRING(qt.text, (qs.statement_start_offset / 2) + 1, 100) AS Query
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
WHERE database_id = DB_ID()
ORDER BY qs.total_elapsed_time DESC;

-- 5. DISK I/O BY DATABASE
SELECT
    DB_NAME(ivfs.database_id) AS DatabaseName,
    ivfs.file_id,
    ivfs.io_stall_read_ms,
    ivfs.io_stall_write_ms,
    ivfs.num_of_reads,
    ivfs.num_of_writes,
    ivfs.file_handle
FROM sys.dm_io_virtual_file_stats(NULL, NULL) ivfs
WHERE database_id NOT IN (1, 2, 3, 4)  -- Exclude system DBs
ORDER BY io_stall_read_ms + io_stall_write_ms DESC;

-- =================================================================
-- INDEX FRAGMENTATION MONITORING
-- =================================================================

-- 6. INDEX FRAGMENTATION REPORT
SELECT
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent,
    ips.index_type_desc,
    ips.page_count,
    CASE
        WHEN ips.avg_fragmentation_in_percent < 10 THEN 'OK'
        WHEN ips.avg_fragmentation_in_percent < 20 THEN 'REORGANIZE'
        ELSE 'REBUILD'
    END AS Action
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
AND ips.page_count > 1000  -- Only significant indexes
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- =================================================================
-- EXTENDED EVENTS SETUP (Slow Queries)
-- =================================================================

-- 7. CREATE EXTENDED EVENT SESSION
CREATE EVENT SESSION SlowQueries ON SERVER
ADD EVENT sqlserver.sql_statement_completed(
    WHERE duration > 1000000  -- > 1 second (in microseconds)
)
ADD TARGET package0.event_file(
    SET filename = N'C:\Traces\SlowQueries.xel',
    max_file_size = 50  -- 50 MB per file
);

ALTER EVENT SESSION SlowQueries ON SERVER STATE = START;

-- 8. QUERY EXTENDED EVENT DATA
SELECT
    event_data.value('(//event/@name)[1]', 'VARCHAR(100)') AS EventName,
    event_data.value('(//event/@timestamp)[1]', 'DATETIME2') AS EventTime,
    event_data.value('(//action[@name="tsql_text"]/value)[1]', 'VARCHAR(MAX)') AS QueryText,
    event_data.value('(//data[@name="duration"]/value)[1]', 'BIGINT') / 1000000.0 AS DurationSeconds,
    event_data.value('(//data[@name="cpu_time"]/value)[1]', 'BIGINT') / 1000000.0 AS CPUTimeSeconds,
    event_data.value('(//data[@name="logical_reads"]/value)[1]', 'BIGINT') AS LogicalReads
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file('C:\Traces\SlowQueries*.xel', NULL, NULL, NULL)
) xel_data
ORDER BY EventTime DESC;

-- =================================================================
-- STATISTICS MONITORING
-- =================================================================

-- 9. STATISTICS AGE (Last Updated)
SELECT
    OBJECT_NAME(s.object_id) AS TableName,
    s.name AS StatisticName,
    STATS_DATE(s.object_id, s.stats_id) AS LastUpdated,
    DATEDIFF(day, STATS_DATE(s.object_id, s.stats_id), GETDATE()) AS DaysSinceUpdate
FROM sys.stats s
WHERE s.object_id IN (SELECT object_id FROM sys.tables WHERE type = 'U')
ORDER BY LastUpdated ASC;

-- 10. UPDATE STATISTICS (All Tables in Database)
--  Based on Auto-Update Tracking
UPDATE STATISTICS;  -- All tables
```

#### Extended Events Session Setup (PowerShell)

```powershell
# ==================================================================
# PowerShell: Extended Events Session Management
# ==================================================================

param(
    [string]$SQLInstance = "[ServerInstance]",
    [string]$EventName = "SlowQueries",
    [int]$DurationThresholdMs = 1000,
    [string]$TracePath = "C:\Traces"
)

# Import SQL module
Import-Module SqlServer -ErrorAction Stop

# Connect to SQL instance
$conn = New-Object System.Data.SqlClient.SqlConnection
$conn.ConnectionString = "Server=$SQLInstance;Integrated Security=true;"
$conn.Open()

# 1. Create Extended Event Session (Slow Queries > 1 sec)
$sql = @"
IF EXISTS (SELECT * FROM sys.server_event_sessions WHERE name = 'SlowQueries_Extended')
    DROP EVENT SESSION SlowQueries_Extended ON SERVER;

CREATE EVENT SESSION SlowQueries_Extended ON SERVER
ADD EVENT sqlserver.sql_statement_completed(
    WHERE duration > $($DurationThresholdMs * 1000)  -- Convert to microseconds
)
ADD EVENT sqlserver.lock_deadlock(
    WHERE database_id = DB_ID()
)
ADD TARGET package0.event_file(
    SET filename = N'$TracePath\SlowQueries.xel',
    max_file_size = 50,
    max_rollover_files = 10
)
WITH (
    STARTUP_STATE = ON,
    TRACK_CAUSALITY = ON
);

ALTER EVENT SESSION SlowQueries_Extended ON SERVER STATE = START;
"@

$cmd = $conn.CreateCommand()
$cmd.CommandText = $sql
$cmd.ExecuteNonQuery() | Out-Null
Write-Host "✓ Extended Event Session 'SlowQueries_Extended' created"

# 2. Query Extended Event Data
$queryEventSql = @"
SELECT TOP 100
    event_data.value('(//event/@timestamp)[1]', 'DATETIME2') AS EventTime,
    event_data.value('(//action[@name="tsql_text"]/value)[1]', 'VARCHAR(MAX)') AS QueryText,
    event_data.value('(//data[@name="duration"]/value)[1]', 'BIGINT') / 1000000.0 AS DurationSeconds
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file(N'$TracePath\SlowQueries*.xel', NULL, NULL, NULL)
) xel_data
ORDER BY EventTime DESC;
"@

$cmd.CommandText = $queryEventSql
$reader = $cmd.ExecuteReader()

$slowQueries = @()
while ($reader.Read()) {
    $slowQueries += [PSCustomObject]@{
        EventTime = $reader["EventTime"]
        QueryText = $reader["QueryText"].Substring(0, [Math]::Min(100, $reader["QueryText"].Length))
        DurationSeconds = $reader["DurationSeconds"]
    }
}

Write-Host "`nSlow Queries:"
$slowQueries | Format-Table -AutoSize

$conn.Close()
```

#### Maintenance Automation (SQL Agent Job)

```sql
-- =================================================================
-- SQL AGENT JOB: Weekly Index Maintenance
-- =================================================================

-- Create job
USE msdb;

EXEC dbo.sp_add_job
    @job_name = N'DBA_WeeklyIndexMaintenance',
    @enabled = 1,
    @description = N'Rebuild/Reorganize fragmented indexes weekly';

-- Step 1: Check Fragmentation
EXEC sp_add_jobstep
    @job_name = N'DBA_WeeklyIndexMaintenance',
    @step_name = N'Step1_CheckFragmentation',
    @subsystem = N'TSQL',
    @database_name = N'YourDatabase',
    @command = N'
        -- REORGANIZE indexes with 10-20% fragmentation
        DECLARE @ObjectID INT, @IndexID INT;

        DECLARE frag_cursor CURSOR FOR
            SELECT ips.object_id, ips.index_id
            FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, ''LIMITED'') ips
            WHERE ips.avg_fragmentation_in_percent BETWEEN 10 AND 20
            AND ips.page_count > 1000;

        OPEN frag_cursor;

        FETCH NEXT FROM frag_cursor INTO @ObjectID, @IndexID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
            DECLARE @IndexName NVARCHAR(128) = 
                (SELECT name FROM sys.indexes WHERE object_id = @ObjectID AND index_id = @IndexID);
            DECLARE @TableName NVARCHAR(128) = OBJECT_NAME(@ObjectID);

            ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REORGANIZE;
            PRINT ''Reorganized: '' + @TableName + ''.'' + @IndexName;

            FETCH NEXT FROM frag_cursor INTO @ObjectID, @IndexID;
        END

        CLOSE frag_cursor;
        DEALLOCATE frag_cursor;
    ',
    @retry_attempts = 2,
    @retry_interval = 5;

-- Step 2: REBUILD indexes with > 20% fragmentation
EXEC sp_add_jobstep
    @job_name = N'DBA_WeeklyIndexMaintenance',
    @step_name = N'Step2_RebuildIndexes',
    @subsystem = N'TSQL',
    @database_name = N'YourDatabase',
    @command = N'
        DECLARE @ObjectID INT, @IndexID INT;

        DECLARE frag_cursor CURSOR FOR
            SELECT ips.object_id, ips.index_id
            FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, ''LIMITED'') ips
            WHERE ips.avg_fragmentation_in_percent > 20
            AND ips.page_count > 1000;

        OPEN frag_cursor;

        FETCH NEXT FROM frag_cursor INTO @ObjectID, @IndexID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
            DECLARE @IndexName NVARCHAR(128) = 
                (SELECT name FROM sys.indexes WHERE object_id = @ObjectID AND index_id = @IndexID);
            DECLARE @TableName NVARCHAR(128) = OBJECT_NAME(@ObjectID);

            ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] 
            REBUILD WITH (FILLFACTOR = 80, ONLINE = ON);
            PRINT ''Rebuilt: '' + @TableName + ''.'' + @IndexName;

            FETCH NEXT FROM frag_cursor INTO @ObjectID, @IndexID;
        END

        CLOSE frag_cursor;
        DEALLOCATE frag_cursor;
    ',
    @retry_attempts = 2,
    @retry_interval = 5;

-- Step 3: Update Statistics
EXEC sp_add_jobstep
    @job_name = N'DBA_WeeklyIndexMaintenance',
    @step_name = N'Step3_UpdateStatistics',
    @subsystem = N'TSQL',
    @database_name = N'YourDatabase',
    @command = N'UPDATE STATISTICS',
    @retry_attempts = 2,
    @retry_interval = 5;

-- Schedule: Saturday 2 AM
EXEC sp_add_schedule
    @schedule_name = N'Saturday_2AM',
    @freq_type = 8,  -- Weekly
    @freq_interval = 64,  -- Saturday
    @active_start_time = 020000;

-- Attach schedule to job
EXEC sp_attach_schedule
    @job_name = N'DBA_WeeklyIndexMaintenance',
    @schedule_name = N'Saturday_2AM';
```

---

### ASCII Diagrams

#### Performance Troubleshooting Flowchart

```
Performance Issue Reported
  │
  ├─ Step 1: Baseline Comparison
  │  ├─ CPU: 25% -> 85% (elevated)
  │  ├─ Memory: 60% -> 90% (elevated)
  │  └─ Disk I/O: 100 IOPS -> 500 IOPS (elevated)
  │
  ├─ Step 2: Identify Bottleneck
  │  ├─ Query sys.dm_os_wait_stats
  │  ├─ Result: PAGEIOLATCH (40%) + LCK_M (35%) + CPU_QUEUE (25%)
  │  └─ Primary Bottleneck: PAGEIOLATCH (disk I/O)
  │
  ├─ Step 3: Investigate Disk I/O
  │  ├─ Query sys.dm_io_virtual_file_stats
  │  ├─ Find: io_stall_read_ms = 5000 sec (very high)
  │  └─ Culprit: Data files on slow disk
  │
  ├─ Step 4: Identify Problematic Queries
  │  ├─ Query sys.dm_exec_query_stats
  │  │  └─ Sort by: total_elapsed_time DESC
  │  └─ Results:
  │     ├─ Query A: SELECT * FROM Customers (15 sec, 100K runs)
  │     ├─ Query B: SELECT * FROM Orders (3 sec, 50K runs)
  │     └─ Query C: Report (2 sec, 100 runs) [expected]
  │
  ├─ Step 5: Root Cause Analysis (Query A)
  │  ├─ Check execution plan
  │  ├─ Index seek? → Good
  │  ├─ Table scan? → Missing index or filter issue
  │  ├─ Join? → Statistics stale? Many rows?
  │  └─ Result: TABLE SCAN on Customers (1M rows, should be index seek)
  │
  ├─ Step 6: Deploy Fix
  │  ├─ Create index: CREATE INDEX idx_Customers_Region ON Customers(Region)
  │  └─ Replan: Query A now uses index (0.5 sec!)
  │
  └─ Step 7: Verify Resolution
     ├─ CPU: 85% -> 30% ✓
     ├─ Disk I/O: 500 IOPS -> 120 IOPS ✓
     ├─ Query A: 15 sec -> 0.5 sec ✓
     └─ Issue Resolved!
```

#### DMV-Based Diagnostic Workflow

```
Connection Established
  │
  ▼
User Executes Query
  │
  ├─ Query In Execution
  │  ├─ sys.dm_exec_requests shows session_id=45, status='running'
  │  ├─ cpu_time = 2000 ms (2 sec so far)
  │  ├─ logical_reads = 50000 (many reads)
  │  └─ Status: RUNNING
  │
  │ (User complains: "Query is slow!")
  │
  ├─ DBA Queries DMVs
  │  ├─ sys.dm_exec_requests (active queries)
  │  │  └─ Session 45: SELECT * FROM Orders (WHERE Region='East')
  │  │
  │  ├─ sys.dm_exec_query_plan (execution plan)
  │  │  └─ Clustered Index Scan (not ideal; table scan)
  │  │
  │  └─ sys.dm_os_wait_stats (cumulative waits)
  │     └─ PAGEIOLATCH (disk I/O) is 60% of total waits
  │
  ├─ Root Cause: INDEX MISSING
  │  └─ Should have index on Region column
  │
  ├─ Solution: CREATE INDEX
  │  └─ CREATE INDEX idx_Orders_Region ON Orders(Region, OrderAmount)
  │
  └─ Verification
     ├─ Query re-runs (new index exists)
     ├─ sys.dm_exec_requests: cpu_time = 200 ms (10x faster!)
     ├─ sys.dm_exec_query_stats: avg_elapsed_time reduced
     └─ ✓ Issue Resolved
```

---

## Hands-on Scenarios

Hands-on scenarios provide realistic DBA situations requiring diagnosis, implementation, and remediation. These are typical situations encountered in production environments.

### Scenario 1: Designing Hybrid HA/DR for 99.99% SLA (4 Nines)

#### Problem Statement

Your organization's flagship financial transaction database (Order Management System) must meet a **99.99% SLA** (max 52 minutes downtime/year). Current setup: Single-server on-premises with nightly backups. The database is **2 TB, 50,000 transactions/minute peak, OLTP**. You must design and implement a hybrid HA/DR solution supporting:

- Local failover < 5 seconds (within same datacenter)
- Regional failover < 1 minute (different geographic region)
- Zero data loss for local failover; RPO ≤ 15 minutes for regional

#### Database Architecture Context

```
Current State (Single Server)
  └─ Server: PROD-SQL-01 (2 TB, 50K tpm)
     ├─ Database: OrderManagement
     ├─ Backup: Nightly (runs 2-3 hours, overlaps peak traffic)
     ├─ No HA/DR
     └─ SLA: 99% (87 hours downtime/year possible)

Target State (Hybrid HA/DR)
  ├─ Local Availability (Windows Cluster, same building)
  │  ├─ PROD-SQL-AG-01 (Primary, sync commit)
  │  ├─ PROD-SQL-AG-02 (Secondary, sync commit)
  │  └─ Listener: OrderMgmt-Listener-Local
  │
  ├─ Regional Availability (Async, different geography)
  │  ├─ REMOTE-SQL-AG-03 (Async secondary)
  │  └─ Listener: OrderMgmt-Listener-Remote
  │
  └─ Offsite Backup (Insurance)
     └─ Quarterly full + transaction log backups to cloud
```

#### Step-by-Step Implementation

**Step 1: Pre-Implementation Assessment (Week 1)**

```sql
-- Verify prerequisites
-- 1. Windows Server Failover Cluster exists
Get-Cluster | Select-Object Name, State, Nodes

-- 2. SQL Server instances are Enterprise (required for AlwaysOn)
SELECT @@VERSION;
SELECT SERVERPROPERTY('Edition');  -- Must be Enterprise Edition

-- 3. Database in FULL recovery model
SELECT name, recovery_model_desc FROM sys.databases 
WHERE name = 'OrderManagement';
-- If not FULL:
ALTER DATABASE OrderManagement SET RECOVERY FULL;

-- 4. Create backup and take full backup (required for AG)
BACKUP DATABASE OrderManagement TO DISK = 'D:\Backups\OrderManagement_full.bak';
BACKUP LOG OrderManagement TO DISK = 'D:\Backups\OrderManagement_log.bak';
```

**Step 2: Enable AlwaysOn Availability Groups (Week 1)**

```sql
-- On primary server (PROD-SQL-AG-01)
-- Enable AlwaysOn feature
ALTER SERVER CONFIGURATION SET HADR ON;
RESTART SQL SERVER;  -- Requires restart

-- Verify
SELECT SERVERPROPERTY('IsHadrEnabled');  -- Should return 1
```

**Step 3: Create Availability Group (Week 2)**

```sql
-- On primary replica (PROD-SQL-AG-01)
CREATE AVAILABILITY GROUP OrderMgmt_AG
WITH (
    NAME = N'OrderMgmt_AG',
    AUTOMATED_BACKUP_PREFERENCE = SECONDARY,
    DB_FAILOVER = ON,  -- Automatic failover on database failures
    DTC_SUPPORT = NONE
)
FOR DATABASE OrderManagement
REPLICA ON
    N'PROD-SQL-AG-01' WITH (
        ENDPOINT_URL = N'TCP://prod-sql-ag-01.contoso.com:5022',
        FAILOVER_MODE = AUTOMATIC,
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        SEEDING_MODE = AUTOMATIC
    ),
    N'PROD-SQL-AG-02' WITH (
        ENDPOINT_URL = N'TCP://prod-sql-ag-02.contoso.com:5022',
        FAILOVER_MODE = AUTOMATIC,
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        SEEDING_MODE = AUTOMATIC
    ),
    N'REMOTE-SQL-AG-03' WITH (
        ENDPOINT_URL = N'TCP://remote-sql-ag-03.azure.com:5022',
        FAILOVER_MODE = MANUAL,
        AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,
        SEEDING_MODE = AUTOMATIC
    );

-- Create listener for local (synchronous) replicas
CREATE AVAILABILITY GROUP LISTENER OrderMgmt_Listener_Local
FOR AVAILABILITY GROUP OrderMgmt_AG
WITH (LISTENER_IP = (
    N'192.168.1.200', N'255.255.255.0'  -- Local network
), PORT = 1433, DHCP = OFF);

-- Create listener for remote (async) replica (optional, for reporting)
CREATE AVAILABILITY GROUP LISTENER OrderMgmt_Listener_Remote
FOR AVAILABILITY GROUP OrderMgmt_AG
WITH (LISTENER_IP = (
    N'10.0.1.200', N'255.255.255.0'  -- Remote network
), PORT = 1433, DHCP = OFF);
```

**Step 4: Verify Synchronization (Week 2)**

```sql
-- Check AG health
SELECT ag.name AS AvailabilityGroup,
       ar.replica_server_name,
       ars.role_desc,
       ars.synchronization_health_desc,
       ars.synchronization_state_desc
FROM sys.availability_groups ag
INNER JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
WHERE ag.name = 'OrderMgmt_AG';

-- Expected output:
-- PROD-SQL-AG-01: PRIMARY, SYNCHRONIZED, HEALTHY
-- PROD-SQL-AG-02: SECONDARY, SYNCHRONIZED, HEALTHY
-- REMOTE-SQL-AG-03: SECONDARY, SYNCHRONIZED, HEALTHY (after seeding completes)
```

**Step 5: Application Connection String Updates (Week 3)**

```csharp
// Old connection string (single server)
"Server=PROD-SQL-01;Database=OrderManagement;Integrated Security=true;"

// New connection string (using listener)
"Server=OrderMgmt-Listener-Local;Database=OrderManagement;Integrated Security=true;
 Connection Timeout=15;MultipleActiveResultSets=true;
 ApplicationIntent=ReadWrite;"  // For read-write transactions

// For reporting (read-only on secondary)
"Server=OrderMgmt-Listener-Local;Database=OrderManagement;Integrated Security=true;
 Connection Timeout=15;MultipleActiveResultSets=true;
 ApplicationIntent=ReadOnly;"  // Routes to secondary
```

**Step 6: Monitoring & Alerting Setup (Week 3)**

```sql
-- Create alert for AG health issues
-- Alert if synchronized state != "Synchronized"
CREATE ALERT AG_Sync_Health
ON EVENT @EventType = 'WMI Event'
WITH @WmiEventQuery = N'SELECT * FROM MSWinsockNamedPipeConnectionFailure'
ADD NOTIFICATION TYPE = SQL
    TO OPERATOR = 'DBAs'
    USING PAGER = 'N';

-- Monitor sync queue size
-- Alert if > 100 MB (indicates lag)
SELECT
    ar.replica_server_name,
    drs.log_send_queue_size,
    drs.redo_queue_size,
    drs.synchronization_state_desc
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_database_replica_states drs 
    ON ar.replica_id = drs.replica_id
WHERE drs.log_send_queue_size > 100;  -- MB
```

**Step 7: Failover Testing Procedure (Week 4)**

```sql
-- Planned failover (maintenance window; zero data loss)
-- Step 1: Verify all replicas synchronized
SELECT * FROM sys.dm_hadr_database_replica_cluster_states
WHERE is_clustered_user_transaction = 0;  -- No uncommitted transactions

-- Step 2: Execute failover to secondary
ALTER AVAILABILITY GROUP OrderMgmt_AG FAILOVER;

-- Step 3: Verify new primary
SELECT replica_server_name, role_desc 
FROM sys.dm_hadr_availability_replica_states
WHERE availability_group_id = (SELECT group_id FROM sys.availability_groups 
                               WHERE name = 'OrderMgmt_AG');
-- Expected: PROD-SQL-AG-02 is now PRIMARY

-- Step 4: Verify application connections redirected
-- Application connection string automatically reroutes
-- Applications should reconnect within connection timeout (15 seconds)

-- Step 5: Failback to original primary
-- Wait 1 minute for catch-up
-- ALTER AVAILABILITY GROUP OrderMgmt_AG FAILOVER;
```

**Step 8: Maintenance Window Integration (Ongoing)**

```sql
-- Index maintenance while avoiding primary impact
-- Run on secondary; then failover

-- On secondary (PROD-SQL-AG-02):
-- 1. Disable replication synchronization temporarily
ALTER AVAILABILITY GROUP OrderMgmt_AG MODIFY REPLICA ON 'PROD-SQL-AG-02'
WITH (AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT);

-- 2. Run maintenance (index rebuild, stats update)
ALTER INDEX idx_Orders_PK ON dbo.Orders REBUILD
WITH (FILLFACTOR = 80, ONLINE = ON);
UPDATE STATISTICS dbo.Orders;

-- 3. Re-enable sync
ALTER AVAILABILITY GROUP OrderMgmt_AG MODIFY REPLICA ON 'PROD-SQL-AG-02'
WITH (AVAILABILITY_MODE = SYNCHRONOUS_COMMIT);

-- 4. Wait for catch-up, then failover
WHILE (SELECT COUNT(*) FROM sys.dm_hadr_database_replica_states 
       WHERE synchronization_state_desc != 'SYNCHRONIZED') > 0
BEGIN
    WAITFOR DELAY '00:00:05';
END
ALTER AVAILABILITY GROUP OrderMgmt_AG FAILOVER;
```

#### Best Practices Applied

1. **Synchronous for Local** – Same-datacenter, low-latency network → synchronous commit (zero data loss)
2. **Asynchronous for Remote** – Different geography, high-latency → asynchronous (acceptable RPO)
3. **Automatic Failover Local Only** – Prevents accidental failover to remote with data loss potential
4. **Application Listener** – Abstracts replica changes; app reconnects automatically
5. **Dedicated Backup Replica** – AUTOMATED_BACKUP_PREFERENCE = SECONDARY offloads backup load from primary
6. **Maintenance Window Strategy** – Leverage secondary for maintenance; transparent to users
7. **Layered Alerting** – Monitor synchronization state, queue sizes, listener health

---

### Scenario 2: Diagnosing and Resolving AG Failover Delays

#### Problem Statement

Your Always On AG has occasional failover delays: Automatic failover takes **45-120 seconds** instead of expected **< 5 seconds**. Production is experiencing user complaints during these incidents. You need to diagnose root cause and implement fix.

#### Symptoms Observed

```
Timeline of Incident:
  2:15 PM: Database connectivity lost (application detects)
  2:15:30 PM: Failover triggers
  2:16:45 PM: Failover completes (90 seconds delay)
  2:16:46 PM: Application reconnects
  └─ Gap: 90 seconds users cannot access database
     └─ Violates SLA requirement (< 30 seconds)
```

#### Root Cause Analysis & Remediation

**Investigation Step 1: Review Extended Events for Failover Activity**

```sql
-- Create Extended Event to capture AG health checks
CREATE EVENT SESSION AG_Failover_Events ON SERVER
ADD EVENT alwayson_ddl_executed,
ADD EVENT availability_group_listener_state_change,
ADD EVENT availability_replica_state_change
ADD TARGET package0.event_file(
    SET filename = N'C:\Traces\AG_Failover.xel'
);
ALTER EVENT SESSION AG_Failover_Events ON SERVER STATE = START;

-- Query failover timeline
SELECT
    event_data.value('(//event/@timestamp)[1]', 'DATETIME2') AS EventTime,
    event_data.value('(//event/@name)[1]', 'VARCHAR(100)') AS EventName,
    event_data.value('(//data[@name="statement"]/value)[1]', 'VARCHAR(MAX)') AS StatementText
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file(N'C:\Traces\AG_Failover*.xel', NULL, NULL, NULL)
) xel_data
WHERE event_data.value('(//event/@timestamp)[1]', 'DATETIME2') > GETDATE() - 1
ORDER BY EventTime;
```

**Investigation Step 2: Check AG Health Statistics**

```sql
-- Identify slow failovers
SELECT
    ag.name,
    ar.replica_server_name,
    ars.role_desc,
    ars.operational_state_desc,
    ars.recovery_health_desc,
    ars.synchronization_health_desc,
    ars.last_hardened_lsn,
    ars.last_redone_lsn
FROM sys.availability_groups ag
INNER JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
WHERE ag.name = 'OrderMgmt_AG';

-- Expected: Both secondaries show HEALTHY, SYNCHRONIZED
-- Red Flag: recovery_health_desc = 'NOT_HEALTHY'
```

**Root Cause Identified: Long-Running Uncommitted Transaction on Primary**

```
Scenario:
  ├─ Primary has long-running transaction (UNDO scenario)
  ├─ Primary fails
  ├─ Failover logic: "Can I promote secondary?"
  │  └─ Secondary checks: "What's my last synchronized LSN?"
  │  └─ Checks: "Is there uncommitted work on primary?"
  │  └─ Must UNDO any uncommitted work on secondary before promoting
  │  └─ UNDO takes 45-120 seconds (depending on transaction size)
  │
  └─ Problem: User runs 30-minute ETL without CHECKPOINT
     └─ Primary fails during ETL
     └─ Secondary must UNDO 30 minutes of work
     └─ Failover delayed until UNDO completes!
```

**Solution 1: Identify Long-Running Transactions**

```sql
-- Find long-running transactions
SELECT
    r.session_id,
    r.command,
    r.status,
    r.database_id,
    r.cpu_time,
    r.total_elapsed_time,
    r.open_transaction_count,
    DATEDIFF(second, start_time, GETDATE()) AS RunningSeconds,
    t.text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.session_id > 50 
    AND r.open_transaction_count > 0
    AND DATEDIFF(second, r.start_time, GETDATE()) > 60  -- > 60 seconds
ORDER BY RunningSeconds DESC;
```

**Solution 2: Implement ETL Best Practices**

```sql
-- ETL job that respects transaction limits
CREATE PROCEDURE sp_LargeETL_SafeForFailover
    @BatchSize INT = 10000
AS
BEGIN
    DECLARE @RowsProcessed INT = 0, @TotalRows INT = 0;

    -- Get total rows to process
    SELECT @TotalRows = COUNT(*) FROM staging.Orders WHERE processed = 0;

    WHILE @RowsProcessed < @TotalRows
    BEGIN
        BEGIN TRANSACTION;

        -- Process batch (limited size)
        INSERT INTO production.Orders (OrderID, Amount, ...)
        SELECT TOP (@BatchSize) OrderID, Amount, ... 
        FROM staging.Orders 
        WHERE processed = 0
        ORDER BY OrderID;

        UPDATE staging.Orders SET processed = 1 
        WHERE OrderID IN (SELECT OrderID FROM production.Orders 
                         WHERE DateInserted > GETDATE() - 1/24/60);

        COMMIT TRANSACTION;

        -- Checkpoint after each batch
        -- Allows secondary to reduce UNDO scope
        CHECKPOINT;

        SET @RowsProcessed = @RowsProcessed + @BatchSize;

        -- Log progress
        PRINT 'Processed: ' + CAST(@RowsProcessed AS VARCHAR) + ' / ' + 
              CAST(@TotalRows AS VARCHAR);

        -- Give secondary time to harden log
        WAITFOR DELAY '00:00:05';
    END
END;
```

**Solution 3: Monitor Transaction Log Sync Queue**

```sql
-- Alert if sync queue accumulates (indicates potential undo delays)
SELECT
    ar.replica_server_name,
    drs.log_send_queue_size AS SendQueueSizeMB,
    drs.redo_queue_size AS RedoQueueSizeMB,
    CASE 
        WHEN drs.log_send_queue_size > 1000 THEN 'HIGH - Risk of failover delay'
        WHEN drs.log_send_queue_size > 100 THEN 'MEDIUM - Monitor closely'
        ELSE 'NORMAL'
    END AS AlertLevel
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_database_replica_states drs 
    ON ar.replica_id = ars.replica_id
WHERE ar.availability_group_name = 'OrderMgmt_AG';
```

**Solution 4: Configure Failover Health Detection**

```sql
-- Adjust failover health check timeout
ALTER AVAILABILITY GROUP OrderMgmt_AG
MODIFY REPLICA ON 'PROD-SQL-AG-01'
WITH (HEALTH_CHECK_TIMEOUT = 30000);  -- 30 seconds

-- This increases detection time but allows for slower networks
-- Trade-off: Longer time to detect failure vs. false positives
```

#### Best Practices Implemented

1. **Batch Processing with CHECKPOINTs** – Limit transaction scope; reduce UNDO work on failover
2. **Monitor Sync Queue Size** – Alert if > 100 MB (indicates potential failover delays)
3. **Disable SA Account** – Only use for system maintenance; prevents UNDO complexity
4. **Extended Events Capture** – Diagnostic data for root cause analysis
5. **Health Check Tuning** – Balance detection speed vs. false positives

---

### Scenario 3: Security Audit and Remediation - Finding Misconfigured Access

#### Problem Statement

Compliance audit flagged: "Multiple users have `db_owner` role when they should only have minimal permissions". You have 200 database logins across 50 databases. Audit requires remediation within 30 days. You need to:

1. Discover over-permissioned users
2. Determine intended permissions
3. Remediate without breaking applications

#### Discovery Process

**Step 1: Audit Current Permissions**

```sql
-- Find all users with db_owner (should be ~ 5 max)
SELECT
    db_name(dp.database_id) AS DatabaseName,
    dp.name AS PrincipalName,
    dp.type_desc,
    dp.principal_id
FROM sys.server_principals sp
INNER JOIN sys.database_principals dp 
    ON sp.principal_id = sp.principal_id
WHERE dp.role_name = 'db_owner'
ORDER BY DatabaseName, PrincipalName;

-- Find all users in sysadmin (should be ~ 3 max)
SELECT
    sp.name AS LoginName,
    sp.type_desc,
    sp.create_date
FROM sys.server_principals sp
INNER JOIN sys.syslogins sl ON sp.principal_id = sl.sid
WHERE sp.name IN (SELECT member_principal_id FROM sys.server_role_members 
                  WHERE role_principal_id = SUSER_ID('sysadmin'))
ORDER BY LoginName;

-- Export to spreadsheet for audit trail
SELECT 
    'Server: ' + @@SERVERNAME AS ServerName,
    GETDATE() AS AuditDate,
    sp.name AS LoginName,
    CASE WHEN sp.type = 'S' THEN 'SQL' ELSE 'Windows' END AS AuthType,
    DB_NAME(dp.database_id) AS DatabaseName,
    dp.name AS UserName,
    'db_owner' AS CurrentRole,
    'N/A' AS ExpectedRole,  -- To be filled by business owner
    'REMEDIATE' AS Action
FROM sys.server_principals sp
INNER JOIN sys.database_principals dp 
    WHERE dp.is_fixed_role = 1 AND dp.name = 'db_owner';
```

**Step 2: Map Application Data Flow (Identify Actual Permissions Needed)**

```
Application Analysis:
  ├─ OrderProcessing App
  │  ├─ Connects as: [CONTOSO/OrderAppAccount]
  │  ├─ Operations:
  │  │  ├─ SELECT Orders (reads)
  │  │  ├─ INSERT OrderDetails (writes)
  │  │  ├─ UPDATE OrderStatus (writes)
  │  │  └─ EXECUTE sp_ProcessOrder (stored proc)
  │  │
  │  └─ Actual permission needed: db_datawriter + db_datareader
  │     (NOT db_owner!)
  │
  └─ Reporting App
     ├─ Connects as: [CONTOSO/ReportAppAccount]
     ├─ Operations:
     │  └─ SELECT * FROM [All tables]  (read-only)
     └─ Permission needed: db_datareader (NOT db_owner!)
```

**Step 3: Create Audit Report**

```sql
-- Generate remediation report
CREATE TABLE AuditReport (
    DatabaseName NVARCHAR(128),
    UserName NVARCHAR(128),
    CurrentRoles NVARCHAR(MAX),
    AppName NVARCHAR(128),
    RecommendedRole NVARCHAR(128),
    BusinessJustification NVARCHAR(MAX),
    RemediationSQL NVARCHAR(MAX),
    RemediationStatus VARCHAR(20)  -- PENDING, COMPLETED, BLOCKED
);

-- Populate report
INSERT INTO AuditReport
SELECT
    DB_NAME(dp.database_id) AS DatabaseName,
    dp.name AS UserName,
    STRING_AGG(drm.role_name, ', ') AS CurrentRoles,
    'OrderProcessing' AS AppName,
    'db_datareader' AS RecommendedRole,
    'App reads orders and inserts details' AS BusinessJustification,
    'ALTER ROLE [db_owner] DROP MEMBER [OrderAppAccount];
     ALTER ROLE [db_datareader] ADD MEMBER [OrderAppAccount];' AS RemediationSQL,
    'PENDING' AS RemediationStatus
FROM sys.database_principals dp
LEFT JOIN sys.database_role_members drm ON dp.principal_id = drm.member_principal_id
WHERE DB_NAME(dp.database_id) = 'OrderManagement'
GROUP BY dp.database_id, dp.name;

-- Generate remediation script
SELECT RemediationSQL FROM AuditReport 
WHERE RemediationStatus = 'PENDING'
ORDER BY DatabaseName, UserName;
```

**Step 4: Test Remediation (Non-Prod First)**

```sql
-- Test in DEV environment first
-- 1. Take backup
BACKUP DATABASE OrderManagement_DEV 
TO DISK = 'D:\Backups\OrderManagement_DEV_PreRemediation.bak';

-- 2. Apply permission changes
ALTER ROLE [db_owner] DROP MEMBER [OrderAppAccount];
ALTER ROLE [db_datareader] ADD MEMBER [OrderAppAccount];
ALTER ROLE [db_datawriter] ADD MEMBER [OrderAppAccount];

-- 3. Test application functionality
-- Run full application test suite
-- Verify: Orders inserted successfully, queries return data, no access denied errors

-- 4. Review Extended Events for permission failures
SELECT event_data.value('(//data[@name="error_number"]/value)[1]', 'INT') AS ErrorNumber,
       event_data.value('(//action[@name="sql_text"]/value)[1]', 'VARCHAR(MAX)') AS QueryText
FROM sys.fn_xe_file_target_read_file('C:\Traces\PermissionAudit*.xel', NULL, NULL, NULL) xel_data
WHERE event_data.value('(//data[@name="success"]/value)[1]', 'INT') = 0;

-- 5. If all tests pass, schedule production remediation
```

**Step 5: Production Remediation (Change Control)**

```sql
-- Production remediation (during maintenance window)
-- Executed with full logging and rollback capability

BEGIN TRANSACTION;

-- 1. Remove from db_owner
ALTER ROLE [db_owner] DROP MEMBER [OrderAppAccount];
ALTER ROLE [db_owner] DROP MEMBER [ReportingAppAccount];
ALTER ROLE [db_owner] DROP MEMBER [DeveloperUser123];

-- 2. Add appropriate roles
ALTER ROLE [db_datareader] ADD MEMBER [OrderAppAccount];
ALTER ROLE [db_datawriter] ADD MEMBER [OrderAppAccount];
ALTER ROLE [db_datareader] ADD MEMBER [ReportingAppAccount];

-- 3. Grant specific object permissions (as needed)
GRANT SELECT ON dbo.Orders TO [ReportingAppAccount];
GRANT SELECT ON dbo.CustomerSensitiveData TO [ReportingAppAccount];
DENY UPDATE ON dbo.CustomerSensitiveData TO [ReportingAppAccount];

-- 4. Verify (before commit)
SELECT dp.name, STRING_AGG(drm.role_name, ', ')
FROM sys.database_principals dp
LEFT JOIN sys.database_role_members drm ON dp.principal_id = drm.member_principal_id
WHERE dp.name IN ('OrderAppAccount', 'ReportingAppAccount', 'DeveloperUser123')
GROUP BY dp.name;

-- If verification good: COMMIT;
-- If issues found: ROLLBACK;
COMMIT;

-- 5. Post-remediation audit
SELECT COUNT(*) AS DbOwnerCount FROM sys.database_role_members 
WHERE role_principal_id = (SELECT principal_id FROM sys.database_principals 
                          WHERE name = 'db_owner');
-- Should return: 1 or 2 (only legitimate DBAs)
```

#### Best Practices Applied

1. **Audit Before Remediation** – Document baseline for compliance requirement
2. **Test in Non-Prod** – Verify application functionality before touching production
3. **Principle of Least Privilege** – Remove excessive permissions; grant only necessary role
4. **Change Control** – Transaction-based remediation with rollback capability
5. **Extended Events Capture** – Monitor for permission failures post-remediation
6. **Business Owner Sign-Off** – Document approval for each user's role change

---

### Scenario 4: Performance Degradation Investigation – Post-Deployment

#### Problem Statement

After code deployment Wednesday 10 PM, user reports: "Critical batch job now takes **90 minutes** (was 15 minutes)". Department cannot proceed with end-of-month close. You have 2 hours to diagnose and resolve. No time for lengthy analysis!

#### Rapid Investigation Framework

```
Timeline:
  10:00 PM: Deployment completes
  10:15 PM: Batch job starts (pre-scheduled)
  10:30 PM: User reports "Still running... normally done by 10:15!"
  10:45 PM: DBA alerted; 2-hour window to resolve
  12:45 AM: Hard deadline (end-of-month cutoff)
```

**Step 1: Establish Baseline (Immediate)**

```sql
-- Capture current running query
SELECT TOP 5
    r.session_id,
    r.command,
    DATEDIFF(minute, r.start_time, GETDATE()) AS RunningMinutes,
    r.cpu_time / 1000 AS CPUTimeSeconds,
    r.total_elapsed_time / 1000 AS ElapsedSeconds,
    r.reads,
    SUBSTRING(t.text, 1, 100) AS Query
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.session_id NOT IN (1, 2, 3, 4)  -- Exclude system
ORDER BY r.total_elapsed_time DESC;

-- Query now at 45 minutes (expected: 15 minutes)
-- Red flag: HIGH IO reads (indicates table scans)
```

**Step 2: Compare Execution Plans (Minute 5)**

```sql
-- Get execution plan from cache
SELECT
    qt.text,
    qs.execution_count,
    CAST(qs.total_elapsed_time / 1000000 / NULLIF(qs.execution_count, 0) AS NUMERIC(10,2)) AS AvgElapsedSeconds,
    query_plan
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) qp
WHERE qt.text LIKE '%ClosingRecon%'
ORDER BY qs.total_elapsed_time DESC
OPTION (RECOMPILE);

-- Check: Is plan changed?
-- Expected: Index seek
-- Actual: Clustered Index Scan
-- Causing: 6x performance degradation
```

**Root Cause Identified (Minute 10)**

```
Code Change Review (Git Diff from Deployment):
  ├─ File: sp_ClosingRecon.sql
  ├─ Line 42: WHERE clause modified
  │  │
  │  ├─ OLD: WHERE RegionID = @Region AND Year = @Year
  │  │       └─ Uses index: idx_RegionYear
  │  │
  │  └─ NEW: WHERE RegionID = @Region 
  │          AND YEAR(TransactionDate) = @Year
  │          └─ YEAR() function prevents index seek! Forces scan.
  │
  └─ Problem: Function in WHERE clause disables index usage
```

**Step 3: Develop Fix (Minute 15)**

**Option 1: Revert Code (3 minutes, zero-risk)**

```sql
-- Fastest option if acceptable to business
-- Roll back to previous deployment
-- sp_ClosingRecon reverted to use Year column directly
```

**Option 2: Implement Index Hint (10 minutes)**

```sql
-- If must keep new code, use HINT
-- File: sp_ClosingRecon.sql

-- OLD (broken):
SELECT * FROM ProductionData
WHERE RegionID = @Region 
  AND YEAR(TransactionDate) = @Year;

-- NEW (with workaround):
SELECT * FROM ProductionData
WHERE RegionID = @Region 
  AND TransactionDate >= DATEFROMPARTS(@Year, 1, 1)
  AND TransactionDate < DATEFROMPARTS(@Year + 1, 1, 1)  -- Date range instead of YEAR()
  -- Allows index on (RegionID, TransactionDate) to be used
;

-- Or if must use YEAR():
SELECT * FROM ProductionData 
WITH (INDEX (idx_RegionYear))  -- Force index hint
WHERE RegionID = @Region 
  AND YEAR(TransactionDate) = @Year;
```

**Option 3: Create Missing Index (5 minutes)**

```sql
-- If no index exists, create one
CREATE INDEX idx_ClosingRecon_Fast
ON ProductionData(RegionID, TransactionDate)
INCLUDE (Amount, Description);

-- Recalculate plan (index now available)
-- Query should now use seek (not scan)
```

**Step 4: Verify Fix (Minute 25)**

```sql
-- Execute sp_ClosingRecon with fix applied
DECLARE @StartTime DATETIME = GETDATE();

EXEC sp_ClosingRecon 
    @Region = 'East',
    @Year = 2024;

DECLARE @EndTime DATETIME = GETDATE();
SELECT DATEDIFF(second, @StartTime, @EndTime) AS ExecutionSeconds;

-- Expected: ~ 15 seconds (back to normal)
-- Result: ✓ FIXED
```

**Step 5: Implement Production (Minute 30)**

```sql
-- Option chosen: Date range query (minimal code change)
-- Update sp_ClosingRecon in production

-- 1. Deploy new version using existing date index
-- 2. Execute batch job again
-- 3. Runs in 16 minutes (success!)
-- 4. End-of-month close proceeds on schedule!
```

#### Post-Incident Review

```
Root Cause: Function in WHERE clause disabled index
   ├─ YEAR(TransactionDate) = @Year → Table scan
   └─ TransactionDate >= ... AND < ... → Index seek

Prevention: 
   ├─ Code review checklist: Avoid functions in WHERE predicates
   ├─ Automated plan analysis: Alert if plan changes > 10%
   └─ A/B testing: Compare query plans before and after deployment for high-impact procs
```

---

### Scenario 5: Disaster Recovery Drill – Activation

#### Problem Statement

Annual disaster recovery drill: **Failover to secondary datacenter and validate that all systems are operational**. You have 2-hour window. Drill objectives:

1. Activate secondary AG replica
2. Verify application connectivity
3. Test backup restoration process
4. Validate data integrity
5. Bring primary back online and resync

#### Drill Execution

**Phase 1: Pre-Drill Preparation (Day Before)**

```sql
-- Verify secondary replica is healthy
SELECT
    ar.replica_server_name,
    ars.role_desc,
    ars.synchronization_state_desc,
    ars.synchronization_health_desc,
    ars.connected_state_desc
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
WHERE ar.availability_group_name = 'OrderMgmt_AG';

-- Expected:
-- Primary: SYNCHRONIZED, HEALTHY
-- Secondary: SYNCHRONIZED, HEALTHY
-- Remote: SYNCHRONIZED, HEALTHY

-- Create snapshot for quick restoration after drill
SELECT *, 'Pre_Drill_Snapshot' AS Snapshot
FROM ProductionData
WHERE CAST(GETDATE() AS DATE) = '2024-03-28';
```

**Phase 2: Execute Failover (Minute 0)**

```sql
-- On Primary:
-- Announce to users: "DR Drill starting; expect 2-minute interruption"

-- Wait for any active transactions to complete
CHECKPOINT;

-- Execute manual failover to secondary
ALTER AVAILABILITY GROUP OrderMgmt_AG FAILOVER;

-- Verify failover completed
SELECT ar.replica_server_name, ars.role_desc
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
WHERE ar.availability_group_name = 'OrderMgmt_AG';

-- Expected:
-- REMOTE-SQL-AG-03: NEW PRIMARY
-- PROD-SQL-AG-01, PROD-SQL-AG-02: SECONDARY
```

**Phase 3: Application Connection Test (Minute 5)**

```csharp
// Test application connection to secondary (new primary)
using (SqlConnection conn = new SqlConnection(
    "Server=OrderMgmt-Listener-Remote;Database=OrderManagement;Integrated Security=true;Connection Timeout=15;"))
{
    conn.Open();
    
    // Verify basic operations work
    SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Orders", conn);
    int orderCount = (int)cmd.ExecuteScalar();
    
    // Expected: Returns row count (connection working)
    Console.WriteLine($"Orders in database: {orderCount}");
    
    // Insert test record
    cmd.CommandText = @"
        INSERT INTO Orders (OrderID, Amount, Status) 
        VALUES (999999, 100.00, 'TEST_DRILL')";
    cmd.ExecuteNonQuery();
    
    // Expected: Insert succeeds (replication working)
    Console.WriteLine("Test insert successful");
}
```

**Phase 4: Data Integrity Check (Minute 15)**

```sql
-- Verify secondary data matches primary expectations
SELECT
    SUM(Amount) AS TotalAmount,
    COUNT(*) AS OrderCount,
    MAX(OrderDate) AS LatestOrder
FROM ProductionData
WHERE OrderDate >= GETDATE() - 30;

-- Compare against pre-drill snapshot
-- Expected: Values match (no data loss)

-- Remove test data
DELETE FROM Orders WHERE OrderID = 999999;
```

**Phase 5: Test Backup Restoration (Minute 30)**

```sql
-- From secondary, take a backup (simulates new primary backup)
BACKUP DATABASE OrderManagement 
TO DISK = 'D:\Backups\OrderManagement_Disaster_20240328.bak';

-- Restore to isolated server for verification
-- (Simulates recovery in separate environment)
RESTORE DATABASE OrderManagement_Verify 
FROM DISK = 'D:\Backups\OrderManagement_Disaster_20240328.bak'
WITH MOVE 'OrderManagement' TO 'E:\Verify\OrderManagement.mdf',
     MOVE 'OrderManagement_log' TO 'E:\Verify\OrderManagement_log.ldf';

-- Verify restored database
SELECT COUNT(*) AS VerifyOrderCount FROM OrderManagement_Verify.dbo.Orders;
-- Expected: Same as current count

-- Cleanup
DROP DATABASE OrderManagement_Verify;
```

**Phase 6: Failback to Primary (Minute 60)**

```sql
-- After successful drill, failback to original primary
-- Step 1: Ensure primary is synced and healthy
-- Step 2: Execute failover
ALTER AVAILABILITY GROUP OrderMgmt_AG FAILOVER;

-- Verify primary is back as PRIMARY
SELECT ar.replica_server_name, ars.role_desc
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
WHERE ar.availability_group_name = 'OrderMgmt_AG';

-- Expected:
-- PROD-SQL-AG-01: PRIMARY
-- REMOTE-SQL-AG-03: SECONDARY
```

**Phase 7: Post-Drill Verification (Minute 90)**

```sql
-- Verify all replicas are healthy and synchronized
SELECT
    ar.replica_server_name,
    ars.role_desc,
    ars.synchronization_state_desc,
    drs.redo_queue_size,
    drs.log_send_queue_size
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id
INNER JOIN sys.dm_hadr_database_replica_states drs
    ON ars.replica_id = drs.replica_id
WHERE ar.availability_group_name = 'OrderMgmt_AG';

-- Expected:
-- All replicas SYNCHRONIZED, HEALTHY, CONNECTED
-- Queue sizes close to 0
```

#### Drill Results & Documentation

```
DR Drill Results - 2024-03-28

✓ Failover completed in 4 seconds
✓ Applications reconnected in 8 seconds
✓ Secondary data integrity verified (no losses)
✓ Backup and restore successful
✓ Failback completed in 3 seconds
✓ All replicas returned to SYNCHRONIZED state

Issues Found:
  ├─ Network latency to secondary: 120ms (higher than expected)
  │  └─ Action: Review network configuration with team
  │
  └─ Backup restore took 25 minutes
     └─ Action: Optimize backup location to faster storage

Lessons Learned:
  ├─ Document all manual steps for faster activation
  ├─ Create runbook for failover procedure
  └─ Quarterly drills required (this found network latency issue)
```

---

## Most Asked Interview Questions

### Question 1: "Explain the quorum voting mechanism in Always On Availability Groups. How does it prevent split-brain scenarios?"

**Expected Answer (Senior DBA):**

---

**Quorum Fundamentals:**

Always On uses **quorum voting** to ensure cluster consensus before failover. In a split-brain scenario (network partition), only one partition can claim the cluster; the other partition is isolated.

**Three Quorum Models:**

```
1. Node Majority (Most Common)
   └─ Voting weight: 1 per node
   └─ Required votes to maintain cluster: > 50%
   └─ Example: 3 nodes
      ├─ Partition A: 2 nodes (wins, > 50%)
      ├─ Partition B: 1 node (loses, < 50%)
      └─ Partition B goes offline, cluster B isolated

2. Node and File Share Witness
   └─ Voting weight: Nodes = 1, Witness = 1
   └─ Example: 2 nodes + 1 file share witness
      ├─ Node 1: 1 vote
      ├─ Node 2: 1 vote
      ├─ Witness: 1 vote (total: 3 votes)
      ├─ Partition A (Node 1): 1 vote (needs > 1.5, fails!)
      ├─ Partition B (Node 2 + Witness): 2 votes (wins)
      └─ Split-brain prevented!

3. Cloud Witness (Azure)
   └─ Voting weight: Nodes + Azure-hosted witness
   └─ Azure witness is trusted arbitrator
```

**Split-Brain Prevention Mechanism:**

```
Healthy Cluster:
  ├─ 3 nodes all connected
  ├─ All vote YES to cluster operations
  └─ Cluster healthy, failover possible

Network Partition (Split-Brain):
  ├─ Partition A: Node 1, Node 2 (2 votes)
  ├─ Partition B: Node 3 (1 vote)  
  │
  └─ Quorum Logic:
     ├─ Partition A: 2 votes > 50% (KEEPS cluster)
     ├─ Partition B: 1 vote = 33% (LOSES cluster, isolated)
     │
     └─ Result:
        ├─ Partition A: Cluster continues, Primary eligible
        ├─ Partition B: Isolated, no cluster operations allowed
        └─ NO SPLIT-BRAIN: Only one partition has quorum!
```

**Why This Matters for Failover:**

```
Scenario: Network partition between datacenters
  ├─ DC-A: Node 1, Node 2, Witness (3 votes)
  ├─ DC-B: Node 3 (1 vote)
  │
  └─ With quorum:
     ├─ DC-A wins: Can promote Primary replica to DC-A
     ├─ DC-B loses: Cannot promote to Primary
     └─ Result: Single Primary guaranteed, no conflicts
```

**Real-World Example (3-Replica AG):**

```sql
-- Create AG with Node Majority quorum
CREATE AVAILABILITY GROUP ProductionAG
  WITH (CLUSTER_TYPE = WSFC, QUORUM_TYPE = NODE_MAJORITY)
  FOR DATABASE ProductionDB
  REPLICA ON
    'Node1' WITH (FAILOVER_MODE = AUTOMATIC),
    'Node2' WITH (FAILOVER_MODE = AUTOMATIC),
    'Node3' WITH (FAILOVER_MODE = MANUAL);

-- Check quorum votes
SELECT
    member_name,
    member_state_desc,
    number_of_quorum_votes
FROM sys.dm_hadr_cluster_members;

-- Expected:
-- Node1: UP, 1 vote
-- Node2: UP, 1 vote
-- Node3: UP, 1 vote
-- Total: 3 votes, Majority: 2
```

---

### Question 2: "You must achieve 99.99% SLA (52 minutes downtime/year) for a 2 TB OLTP database. Design the architecture."

**Expected Answer (Senior DBA - Architectural Reasoning):**

---

**SLA Math:**

```
99.99% = 52.56 minutes downtime allowed per year
  ├─ RPO Requirement: 0 data loss (financial transactions)
  ├─ RTO Requirement: < 5 minutes (business critical)
  └─ Availability Window: 24/7 (no maintenance window)
```

**Proposed Architecture (Layered Redundancy):**

```
Primary Datacenter (Local HA)
├─ SQL Server 1 (Primary, Always On, Sync Commit)
├─ SQL Server 2 (Secondary, Always On, Sync Commit)
├─ SQL Server 3 (Tertiary, Always On, Async Commit)
└─ Listener: OrderMgmt-AG-Local (routes to primary)

Secondary Datacenter (Geographic DR)
├─ SQL Server 4 (Always On Async Replica)
├─ Log Shipping (Backup + Restore from Primary)
└─ Listener: OrderMgmt-AG-Remote

Backup & Offsite
├─ Database backups every 4 hours → Cloud blob storage
├─ Transaction log backups every 15 minutes
└─ Test restore quarterly
```

**Failover Scenarios & RTO/RPO:**

```
Scenario 1: Single Node Failure (Node 1)
  ├─ Detection: 5 seconds (AG listener detects)
  ├─ Failover: Automatic to Node 2
  ├─ Application reconnection: 10 seconds
  ├─ Total RTO: ~15 seconds
  ├─ RPO: 0 (synchronous commit)
  └─ Business impact: Minimal; users experience brief reconnection

Scenario 2: Local Datacenter Power Loss
  ├─ Detection: 30 seconds (health check timeout)
  ├─ Failover: Automatic to Node 3
  ├─ RTO: ~45 seconds
  ├─ RPO: 0 (still synchronous, lag minimal)
  └─ Covered by 99.99% SLA

Scenario 3: Entire Primary Datacenter Failure
  ├─ Detection: 60 seconds (network heartbeat loss)
  ├─ Failover: Manual to Secondary DC (Node 4)
  ├─ RTO: ~5 minutes (includes manual intervention + connection routing)
  ├─ RPO: ~15 minutes (async lag + log backup delay)
  └─ Within acceptable SLA limit
```

**Implementation Details:**

```sql
-- Primary Datacenter AG (Synchronous)
CREATE AVAILABILITY GROUP OrderMgmt_AG
WITH (
    AUTOMATED_BACKUP_PREFERENCE = SECONDARY,
    DB_FAILOVER = ON,
    FAILOVER_CLUSTER_NAME = 'DC-A-Cluster'
)
FOR DATABASE OrderManagement
REPLICA ON
    'PRIMARY-SQL-01' WITH (
        ENDPOINT_URL = 'TCP://primary-sql-01:5022',
        FAILOVER_MODE = AUTOMATIC,
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
    ),
    'SECONDARY-SQL-01' WITH (
        ENDPOINT_URL = 'TCP://secondary-sql-01:5022',
        FAILOVER_MODE = AUTOMATIC,
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
    ),
    'TERTIARY-SQL-01' WITH (
        ENDPOINT_URL = 'TCP://tertiary-sql-01:5022',
        FAILOVER_MODE = AUTOMATIC,
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
    ),
    'REMOTE-DC-SQL-01' WITH (
        ENDPOINT_URL = 'TCP://remote-dc-sql-01:5022',
        FAILOVER_MODE = MANUAL,
        AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT
    );

-- Secondary Datacenter (DR) AG
-- Separate AG with manual failover + Log Shipping backup pipeline
```

**Availability Math:**

```
Node Failure Probability: 99.5% uptime per node/year
├─ Single node: 99.5%
├─ 2 sync nodes (must both fail): 99.5% × 99.5% = 0.9900 = 99.00%
├─ 3 nodes (2 must fail): 99.5% × 99.5% × 99.5% = 98.51%
└─ 3 nodes + manual DC failover: 99.50% (effectively)

Geographic DR (Add secondary datacenter):
├─ If DC-A fails: Failover to DC-B (5 min RTO, within SLA)
├─ 3 nodes in DC-A: 99.50% uptime
├─ 1 node in DC-B: 99.50% uptime
├─ Total: 99.50% + (0.50% × 99.50%) = 99.9975% ≈ 99.99% ✓
```

**Cost/Benefit Analysis:**

```
Option 1: Single Server (99%)
  ├─ Cost: $50K (hardware + software licenses)
  ├─ Downtime/year: 87 hours
  └─ Too risky for financial data

Option 2: Single Failover (99.9%)
  ├─ Cost: $120K (2 servers + cluster)
  ├─ Downtime/year: 9 hours
  └─ Better, but not enough for SLA

Option 3: Hybrid (Local + Regional) [RECOMMENDED]
  ├─ Cost: $300K (4 servers, complex, but justified)
  ├─ Downtime/year: < 1 hour
  ├─ RPO/RTO: 0/5 min
  └─ Meets 99.99% SLA requirement
```

---

### Question 3: "Compare Transactional Replication vs. Always On AG for use case of real-time read-only reporting replica. When would you choose each?"

**Expected Answer (Senior DBA - Technology Selection):**

---

**Comparison Matrix:**

```
┌────────────────┬─────────────────┬──────────────────────┐
│ Characteristic │ AG (Sync)       │ Replication (Tx)     │
├────────────────┼─────────────────┼──────────────────────┤
│ Data Sync      │ Transaction     │ After commit         │
│ Latency        │ < 100 ms        │ 100 ms - 5 sec       │
│ Data Loss      │ Zero (sync)     │ Possible (async)     │
│ Read Replicas  │ Built-in        │ Subscriber database  │
│ Filtering      │ No              │ Yes (article filters)│
│ Scalability    │ Up to 8 replicas│ Many subscribers     │
│ Network Reqs   │ Low latency     │ Can handle WAN       │
│ Overhead       │ Moderate        │ Higher (log reader)  │
│ DDL Changes    │ Automatic       │ Manual + resubscribe │
│ Cost (Licensing)│ Enterprise only │ All editions         │
└────────────────┴─────────────────┴──────────────────────┘
```

**When to Choose AG:**

```
Use Case: Real-time reporting within same datacenter (< 10ms latency)

Scenario:
  ├─ OLTP Primary: Orders table, 50K tpm
  ├─ Reporting Secondary: Same Orders (readable)
  ├─ Requirement: Reports must be current (< 1 sec lag)
  ├─ Network: High-speed LAN (< 1ms)
  └─ Edition: Enterprise (licensing not a constraint)

Implementation:
  CREATE AVAILABILITY GROUP ReportingAG
  FOR DATABASE OrderManagement
  REPLICA ON
    'PROD-SQL-01' WITH (AVAILABILITY_MODE = SYNCHRONOUS_COMMIT),
    'REPORT-SQL-01' WITH (AVAILABILITY_MODE = SYNCHRONOUS_COMMIT);

  -- Application
  -- OLTP: Connect to listener (routes to primary)
  -- Reporting: ApplicationIntent=ReadOnly (routes to secondary)

Advantages:
  ├─ Automatic failover (if primary fails)
  ├─ Zero data loss
  ├─ Transparent to reporting queries
  └─ No subscriber maintenance
```

**When to Choose Replication:**

```
Use Case: Distributed reporting across geographic regions

Scenario:
  ├─ HQ Database: Financial master in DC-A
  ├─ Regional Subscribers: DC-B, DC-C, DC-D (reporting replicas)
  ├─ Requirement: Each region can query reporting DB locally
  ├─ Network: WAN latency 200-500ms (acceptable for filtered data)
  └─ Edition: All supported (licensing cheaper)

Implementation:
  -- Publisher (HQ): OrderManagement database
  -- Create publication (selected articles: Orders, Customers)
  -- Filter: Exclude sensitive columns, partition by region

  EXEC sp_addpublication
    @publication = 'Replication_Regional',
    @snapshot_in_defaultfolder = 1;

  -- Subscribers (DC-B, DC-C, DC-D)
  EXEC sp_addsubscription
    @publication = 'Replication_Regional',
    @subscriber = 'REPORT-SQL-B-01',
    @subscription_type = 'pull',
    SYNC_TYPE = 'automatic';

  -- Each region's reporting queries local subscriber
  -- No network round-trip to HQ

Advantages:
  ├─ Geographic distribution (no WAN round-trip)
  ├─ Selective replication (filter unwanted tables)
  ├─ Possible on Standard Edition
  ├─ Lower cost for multi-region setup
  ├─ Subscriber databases can be modified (transforms, aggregations)
  └─ Good for compliance (data sovereignty, regional copies)
```

**Hybrid Approach (Best of Both):**

```
Implementation:
  ├─ Primary (HQ): Always On AG for local HA/DR
  ├─ Local Secondary: Readable replica (ApplicationIntent=ReadOnly)
  ├─ Regional Subscribers: Replication for WAN distribution
  │
  └─ Data Flow:
     Primary → Secondary (AG sync)
     Primary → Remote Publisher → Subscribers (TX replication)

Benefits:
  ├─ HQ gets zero-data-loss failover (AG sync)
  ├─ Regions get local read replicas (replication)
  ├─ No network round-trip for regional queries
  └─ Compliance: Data stays in region, doesn't centralize
```

---

### Question 4: "Design a log shipping strategy for a 10 TB database where you must support both Point-in-Time recovery and quick switchover. Explain the trade-offs."

**Expected Answer (Senior DBA - Operational Design):**

---

**Log Shipping Architecture:**

```
Primary (Production)
├─ Database: ProductionDB (10 TB)
├─ Backup Job: Every 4 hours (Full backup)
├─ Log Backup Job: Every 15 minutes
└─ Retention: Backups stored for 3 days

Standby Server
├─ Restore Job: Copy log backups, restore with NORECOVERY
├─ Standby Mode: Database in restore mode
├─ Point-in-Time Recovery: Any point in last 3 days
└─ Switchover: Manual activation (database goes online)
```

**Backup Strategy Specifics:**

```sql
-- Primary: Full backup every 4 hours (occurs overnight during low traffic)
BACKUP DATABASE ProductionDB 
TO DISK = 'D:\Backups\ProductionDB_full_20240328_0200.bak'
WITH COMPRESSION, VERIFY_ONLY;

-- Primary: Transaction log backups every 15 minutes
BACKUP LOG ProductionDB 
TO DISK = 'D:\Backups\ProductionDB_log_20240328_1400.trn'
WITH COMPRESSION;

-- Standby: Automated restore pipeline
RESTORE LOG ProductionDB 
FROM DISK = 'D:\LogShipping\ProductionDB_log_20240328_1400.trn'
WITH NORECOVERY;  -- Database remains restoring, ready for more logs
```

**Trade-offs Analysis:**

```
Backup Frequency ↔ Recovery Time Objective (RTO)

Conservative (Fewer backups):
  ├─ Full backup: Daily (1 time)
  ├─ Log backup: Every hour
  ├─ Standby lag: Up to 1 hour behind
  ├─ RTO if switchover needed: ~30 min (apply final logs)
  ├─ Advantage: Less network/disk overhead
  └─ Disadvantage: Longer lag; potential data loss

Aggressive (More backups):
  ├─ Full backup: Every 4 hours
  ├─ Log backup: Every 15 minutes
  ├─ Standby lag: 15-30 minutes behind
  ├─ RTO if switchover needed: ~5 min (minimal logs to apply)
  ├─ Advantage: Near real-time replica; quick switchover
  └─ Disadvantage: Higher network/disk I/O

Point-in-Time Recovery Window ↔ Disk Space

3-Day Retention (Recommended):
  ├─ Daily full backups: 3 × (10 TB compressed) = 30 TB
  ├─ Log backups: 15 min × 96 per day × 3 days = 432 log files
  ├─ Total disk: ~50 TB (accounting for compression, log size)
  ├─ Advantage: Recover from incidents 72 hours old
  └─ Disadvantage: Expensive disk infrastructure

1-Day Retention (Cost-optimized):
  ├─ Disk needed: ~20 TB
  ├─ Advantage: Lower storage cost
  └─ Disadvantage: Can only recover within 24 hours
```

**Recommended Configuration (Balance):**

```sql
-- Primary Backup Jobs
-- 1. Full backup: Every 4 hours (midnight, 4AM, 8AM, noon, etc.)
sql_backup_full_ProductionDB_04H
├─ BACKUP DATABASE ProductionDB 
│  TO DISK = '\\BackupServer\Full\ProductionDB_full_YYYYMMDD_HHMM.bak'
│  WITH COMPRESSION, VERIFY_ONLY;
├─ Schedule: Every 4 hours
└─ Retention: 7 days

-- 2. Log backup: Every 15 minutes
sql_backup_log_ProductionDB_15M
├─ BACKUP LOG ProductionDB 
│  TO DISK = '\\BackupServer\Logs\ProductionDB_log_YYYYMMDD_HHMM.trn'
│  WITH COMPRESSION;
├─ Schedule: Every 15 minutes
└─ Retention: 3 days

-- Standby Jobs
-- 1. Copy backups from share to local disk
copy_backups_to_standby
├─ Robocopy \\BackupServer\Logs \\StandbyServer\LocalBackups
├─ Schedule: Every 15 minutes
└─ Verify: Checksum on copied files

-- 2. Restore process on standby
restore_logs_to_standby
├─ FOR EACH log file in sequence:
│  RESTORE LOG ProductionDB FROM DISK = '...' WITH NORECOVERY;
├─ Schedule: After each log backup (monitor for backlog)
└─ If backlog > 5 backups, alert DBA (copy job lagging)
```

**Point-in-Time Recovery Procedure:**

```sql
-- Example: Recover database to 2 hours ago (bad query deleted data)
-- Current time: 3:00 PM
-- Recovery target: 1:00 PM

-- Step 1: Restore last full backup BEFORE target time
RESTORE DATABASE ProductionDB_Recovery 
FROM DISK = 'D:\Backups\ProductionDB_full_20240328_0000.bak'
WITH MOVE 'ProductionDB' TO 'E:\Recovery\ProductionDB.mdf',
     MOVE 'ProductionDB_log' TO 'E:\Recovery\ProductionDB_log.ldf',
     NORECOVERY;

-- Step 2: Restore logs up to specific time (before bad query)
-- Steps: Apply 0001 thru 0050 logs (covers 12 AM to 1 PM)
RESTORE LOG ProductionDB_Recovery 
FROM DISK = 'D:\Backups\ProductionDB_log_20240328_0100.trn'
WITH NORECOVERY;

-- ... Restore all logs up to 1:00 PM ...

-- Step 3: Restore with STOPAT to freeze at exact time
RESTORE LOG ProductionDB_Recovery 
FROM DISK = 'D:\Backups\ProductionDB_log_20240328_0100.trn'
WITH STOPAT = '2024-03-28 13:00:00', RECOVERY;

-- Step 4: Verify data is correct (before bad query time)
SELECT * FROM ProductionDB_Recovery.dbo.Customers 
WHERE MarkedForDelete = 0;  -- Data preserved!

-- Step 5: Put recovered database online and resume operations
-- (Users accept 2-hour rollback vs. entire day loss)
```

**Rapid Switchover Procedure:**

```sql
-- In failover scenario (primary down):

-- On Standby Server:
-- 1. Stop log restore jobs (if still running)
--    (Prevents waiting for next scheduled backup)

-- 2. Bring database online
ALTER DATABASE ProductionDB SET ONLINE;
-- OR
RESTORE DATABASE ProductionDB WITH RECOVERY;  -- If still in RESTORING mode

-- 3. Verify database is accessible
SELECT COUNT(*) FROM ProductionDB.dbo.Orders;  -- Should work

-- 4. Update connection strings in application
-- "Server=StandbyServer;Database=ProductionDB;..."

-- 5. RTO achieved: 5-10 minutes (minimal final logs + online transition)
--    RPO: Up to 15 minutes behind (lag from last log backup)
```

---

### Question 5: "When would TDE vs. Always Encrypted be your encryption choice? A startup is entering healthcare and must protect PHI (PII: SSN, medical record number). Design the security layer."

**Expected Answer (Senior DBA - Compliance-Driven Encryption):**

---

**Encryption Choice Decision Matrix:**

```
┌──────────────────┬──────────────┬──────────────────┐
│ Criteria         │ TDE          │ Always Encrypted │
├──────────────────┼──────────────┼──────────────────┤
│ Layer            │ At-rest      │ In-use           │
│ DBA Visibility   │ Can read all │ Cannot read PII  │
│ Query Filtering  │ Yes (any col)│ No (deterministic)
│ Performance      │ Minimal      │ Application-side │
│ Encryption Scope │ Whole DB     │ Column-specific  │
│ Key Location     │ On-server    │ Key Vault (best) │
└──────────────────┴──────────────┴──────────────────┘
```

**Healthcare Security Requirements (HIPAA Compliance):**

```
Requirement: Protect PII (SSN, Med Record #)
├─ If database compromised: Encrypted data unreadable
├─ If disk stolen: No plaintext recovery
├─ If DBA has access: Still cannot view encrypted columns
├─ Audit trail: Log who accesses what data
└─ Regulatory: Encryption required for HIPAA certification
```

**Recommended Solution: LAYERED ENCRYPTION**

```
Layer 1: Database Encryption at Rest (TDE)
├─ Encrypts entire database files
├─ Transparent to applications
├─ DBA sees clear text in memory
├─ Protection: Stolen disk drives are encrypted
└─ Implementation: 10 minutes, zero code changes

Layer 2: Sensitive Column Encryption (Always Encrypted)
├─ Encrypts only: SSN, MedRecordNumber, CreditCard
├─ DBA cannot read encrypted columns (good for compliance)
├─ Client application decrypts in memory
├─ Protection: Even if database opened directly, data protected
└─ Implementation: Requires app updates, ~2 weeks

Layer 3: Application-Level Masking (Dynamic Data Masking)
├─ Non-admin users see: SSN = xxx-xx-****
├─ Database shows: Masked for select users
├─ Real SSN: Available only to admin queries
├─ Protection: Prevents accidental disclosure in reports
└─ Implementation: 1 week, SQL-side rules

Layer 4: Auditing & Monitoring
├─ Log all access to sensitive tables
├─ Alert on bulk downloads
├─ Monitor failed access attempts
└─ Implementation: Extended Events, SQL Agent alerts
```

**Implementation Roadmap:**

```
Phase 1: TDE (Week 1 - Immediate)
├─ Create Service Master Key (if not exists)
├─ Create Certificate backed by SMK
├─ Enable encryption on PatientDB
└─ Zero downtime; 2-3% performance impact

Phase 2: Always Encrypted (Week 2-4)
├─ Create Column Master Key in Key Vault
├─ Create Column Encryption Key
├─ Encrypt: Patients.SSN, Patients.MedRecordNumber
├─ Update application to support client-side encryption
└─ Test before production rollout

Phase 3: Dynamic Data Masking (Week 3)
├─ Create masking rule: SSN → xxx-xx-####
├─ Apply to db_datareader role
└─ Admin role sees actual values (needed for billing)
```

**Detailed Implementation (TDE):**

```sql
-- Step 1: Create Service Master Key (if needed)
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Str0ng!Password#2024';

-- Step 2: Create Certificate for TDE
CREATE CERTIFICATE PatientDB_TDE_Cert
WITH SUBJECT = 'Certificate for TDE on PatientDB';

-- Step 3: Enable TDE on database
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE PatientDB_TDE_Cert;

ALTER DATABASE PatientDB
SET ENCRYPTION ON;

-- Step 4: Verify encryption
SELECT
    db_name(database_id) AS DatabaseName,
    encryption_state_desc,
    percent_complete
FROM sys.dm_database_encryption_keys;

-- Expected: Encryption_state_desc = 'Encrypted'
```

**Always Encrypted Implementation (Sensitive Columns):**

```sql
-- Step 1: Create Column Master Key (in Key Vault)
CREATE COLUMN MASTER KEY PatientDB_CMK
WITH (
    KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
    KEY_PATH = 'https://my-keyvault.vault.azure.net/keys/PatientDB-CMK'
);

-- Step 2: Create Column Encryption Key
CREATE COLUMN ENCRYPTION KEY PatientDB_CEK
WITH VALUES (
    COLUMN_MASTER_KEY = PatientDB_CMK,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x...
);

-- Step 3: Encrypt columns (create new encrypted column)
-- Old: Patients.SSN (plaintext)
-- New: Patients.SSN_Encrypted (encrypted)

ALTER TABLE Patients
ADD SSN_Encrypted VARCHAR(11) 
COLLATE Latin1_General_BIN2 
ENCRYPTED WITH (
    ENCRYPTION_TYPE = Randomized,
    ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256',
    COLUMN_ENCRYPTION_KEY = PatientDB_CEK
);

-- Copy data
UPDATE Patients
SET SSN_Encrypted = SSN
WHERE SSN_Encrypted IS NULL;

-- Drop old column
ALTER TABLE Patients DROP COLUMN SSN;

-- Step 4: Migrate application
-- Connection string: Column Encryption Setting = enabled
// C#
string connectionString = "Server=PatientDB;Database=PatientDB;" +
    "Column Encryption Setting=enabled;";
```

**Dynamic Data Masking (Non-Admin View):**

```sql
-- DBA/Admin sees real data
SELECT SSN, MedRecordNumber FROM Patients;
-- Result: 123-45-6789, MR-987654

-- Regular users see masked data
-- (Only if role = db_datareader)

CREATE FUNCTION dbo.fn_SSN_Mask(@SSN VARCHAR(11))
RETURNS TABLE
AS
RETURN SELECT CASE 
    WHEN USER_NAME() IN ('admin', 'dba') THEN @SSN
    ELSE 'xxx-xx-' + RIGHT(@SSN, 4)
END AS SSN_Display;

-- Application query
SELECT dbo.fn_SSN_Mask(SSN) AS SSN FROM Patients;
-- Non-admin result: xxx-xx-6789
-- Admin result: 123-45-6789
```

**Audit Trail (Extended Events):**

```sql
-- Create event session for PII access
CREATE EVENT SESSION AuditPHIAccess ON SERVER
ADD EVENT sqlserver_sql_statement_completed
WHERE SQL_TEXT LIKE '%SSN%' OR SQL_TEXT LIKE '%MedRecordNumber%'
ADD TARGET package0.event_file(
    SET filename = N'\\AuditServer\PHI_Access.xel'
);
ALTER EVENT SESSION AuditPHIAccess ON SERVER STATE = START;

-- Alert on bulk download
SELECT
    event_data.value('(//data[@name="rows"]/value)[1]', 'INT') AS RowsReturned,
    event_data.value('(//action[@name="database_user_name"]/value)[1]', 'VARCHAR(100)') AS UserName,
    event_data.value('(//event/@timestamp)[1]', 'DATETIME2') AS EventTime
FROM sys.fn_xe_file_target_read_file('\\AuditServer\PHI_Access*.xel', NULL, NULL, NULL)
WHERE event_data.value('(//data[@name="rows"]/value)[1]', 'INT') > 1000  -- Alert on large exports
```

**Compliance Checklist:**

```
✓ TDE enabled (database encryption at rest)
✓ Always Encrypted on PII columns (column encryption in-use)
✓ Azure Key Vault integration (keys not on-server)
✓ Dynamic Data Masking for non-admin users
✓ Extended Events capturing PII access
✓ Quarterly authentication review
✓ Annual penetration testing
✓ Incident response plan documented

HIPAA Compliance Achieved:
├─ Data at rest: Encrypted (TDE)
├─ Data in transit: TLS 1.2+ enforced
├─ Data in use: Always Encrypted (client-side)
├─ Access control: RBAC + field-level masking
└─ Audit trail: Complete PHI access logging
```

---

### Question 6: "Walk me through a database corruption scenario. Patient records show 'bad data' in one table. How would you investigate, validate the extent of corruption, and recover safely?"

**Expected Answer (Senior DBA - Incident Response):**

---

**Incident Timeline:**

```
9:00 AM: Report received - "Patient records showing wrong ages"
9:15 AM: Medical staff confirms systematic corruption
10:00 AM: DBA alert - DBCC CHECKDB pending (test server)
10:30 AM: Investigation begins - Prod vs Test comparison
```

**Step 1: Rapid Triage – Assess Severity**

```sql
-- Does CHECKDB detect logical corruption?
DBCC CHECKDB (PatientDB, REPAIR_REBUILD);

-- Output might show:
-- "Msg 8645: Logical consistency error in index 'idx_PatientAge'"
-- "Error: Page 5432 has corrupted data"

-- Extent of corruption: Full table or specific records?
SELECT COUNT(*) AS TotalRows FROM Patients;  -- 1 million
SELECT COUNT(*) AS CorruptedRows
    FROM Patients
    WHERE Age > 150 OR Age < 0;  -- Nonsensical ages
    -- Result: 5,432 corrupted rows (0.5% of DB)

-- Severity: LOW to MODERATE
-- (Not entire table loss, but integrity compromised)
```

**Step 2: Root Cause Investigation**

```
Hypothesis 1: Application Bug
├─ Age calculation: Birth_Year - Current_Year
├─ Check: Source code changes in last deployment (Wednesday)
├─ Git diff: Found typo in age calculation
│  └─ OLD: Age = YEAR(GETDATE()) - YEAR(BirthDate)
│  └─ NEW: Age = BirthDate - YEAR(GETDATE())  <- WRONG!
├─ Timeline: Bug deployed Wednesday 10 PM
├─ Data affected: All records inserted/updated since then
└─ Verdict: Application bug (not hardware failure)

Hypothesis 2: Hardware/Disk Corruption
├─ Would affect random pages
├─ Would show checksum errors in DBCC
├─ DBCC output: No checksum errors detected
└─ Verdict: NOT a hardware issue

Hypothesis 3: Malicious Activity
├─ Check SQL Agent job history: No unauthorized jobs
├─ Check login attempt logs: No suspicious access
├─ Check extended events: Only app account updating data
└─ Verdict: No malicious intent

ROOT CAUSE: Application bug in age calculation
```

**Step 3: Identify Corrupted Data Window**

```sql
-- Define corruption window (when bug existed)
-- Bug deployed: 2024-03-27 10 PM
-- Bug fixed: 2024-03-28 2 AM
-- Corruption window: 4 hours

-- Query last_updated column to find affected records
SELECT
    PatientID,
    Age AS CorruptedAge,
    BirthDate,
    YEAR(GETDATE()) - YEAR(BirthDate) AS CorrectAge,
    last_updated
FROM Patients
WHERE last_updated BETWEEN '2024-03-27 22:00' AND '2024-03-28 02:00'
AND Age > 150;  -- Obviously wrong

-- Results:
-- PatientID 12345: Age 1948 (should be 48)
-- PatientID 54321: Age -2000 (should be 35)
-- Total affected: 5,432 rows
```

**Step 4: Recovery Strategy Decision**

```
Option A: Restore from backup (4-hour data loss)
├─ Last clean backup: 2024-03-27 6 PM
├─ Data loss: 16 hours of patient records (UNACCEPTABLE)
├─ Impact: Medical staff must re-enter all records
└─ Verdict: NOT viable for healthcare

Option B: Restore corrupted table from backup, merge new records (PREFERRED)
├─ Restore Patients table from 2024-03-27 6 PM backup → PatientTable_Clean
├─ Copy new records (inserted after 6 PM, before corruption started 10 PM)
├─ Update corrupted records with calculated fix
├─ Verify against audit trail
└─ Verdict: Minimal data loss, audit trail preserved

Option C: Fix in-place with calculated values (FASTEST)
├─ Calculate correct age: YEAR(GETDATE()) - YEAR(BirthDate)
├─ Update all corrupted records in single transaction
├─ Verify before commit
├─ Rollback if issues found
└─ Verdict: No data loss if calculation correct
```

**Step 5: Execute Recovery (Option C - Fix In-Place)**

```sql
-- Backup before attempting fix (recovery baseline)
BACKUP DATABASE PatientDB 
TO DISK = 'D:\Backups\PatientDB_Pre_Corruption_Fix.bak';

-- Verify backup integrity
RESTORE VERIFYONLY FROM DISK = 'D:\Backups\PatientDB_Pre_Corruption_Fix.bak';

-- BEGIN TRANSACTION (atomic recovery)
BEGIN TRANSACTION CorruptionFix
    SAVE TRANSACTION PreFix;

-- Fix corrupted ages
UPDATE Patients
SET Age = YEAR(GETDATE()) - YEAR(BirthDate)
WHERE PatientID IN (
    SELECT PatientID FROM Patients
    WHERE last_updated BETWEEN '2024-03-27 22:00' AND '2024-03-28 02:00'
    AND (Age > 150 OR Age < 0)
);

-- Verify fix
SELECT
    'Before' AS Phase,
    COUNT(*) AS BadRecords
FROM Patients
WHERE Age > 150 OR Age < 0
UNION ALL
SELECT 'After', 0;  -- Should be 0

-- IF verification successful: COMMIT
-- IF issues found: ROLLBACK TRANSACTION PreFix;
COMMIT TRANSACTION CorruptionFix;

-- Force full backup (after integrity fix)
BACKUP DATABASE PatientDB 
TO DISK = 'D:\Backups\PatientDB_Post_Corruption_Fix.bak'
WITH INIT;  -- Overwrite previous backup set
```

**Step 6: Validation & Audit Trail**

```sql
-- Verify data accuracy against independent source
-- Compare with:
-- 1. Insurance records (external system of record)
-- 2. Manual patient chart (age on admission)
-- 3. Previous backups (spot-check historical records)

SELECT
    p.PatientID,
    p.Age AS DatabaseAge,
    ext.Age AS InsuranceRecordAge,
    CASE 
        WHEN p.Age = ext.Age THEN '✓ MATCH'
        ELSE '✗ MISMATCH - Investigate'
    END AS ValidationStatus
FROM Patients p
LEFT JOIN ExternalInsuranceSystem ext ON p.PatientID = ext.PatientID
WHERE p.last_updated BETWEEN '2024-03-27 22:00' AND '2024-03-28 02:00'
ORDER BY ValidationStatus DESC;

-- Create audit log entry
INSERT INTO DataIntegrityAuditLog (
    DatabaseName, TableName, IncidentDate, CorruptionType,
    RowsAffected, RootCause, RecoveryMethod, ValidatedBy
) VALUES (
    'PatientDB', 'Patients', '2024-03-28', 'Age calculation bug',
    5432, 'Application bug - expression order wrong',
    'Calculated age fix', 'DBA + Medical Records Manager'
);
```

**Step 7: Preventive Measures (Long-term)**

```
1. Code Review Process
   ├─ All stored procedures and calculations reviewed
   ├─ Data validation checks added to audit trail
   └─ Test suite includes data integrity checks

2. Monitoring & Alerting
   ├─ Alert on unusual value ranges (Age > 150)
   ├─ DBCC CHECKDB run daily (detect corruption fast)
   └─ Extended Events capture data anomalies

3. Backup Strategy
   ├─ Point-in-time recovery enabled (transaction logs)
   ├─ 10-day retention of full backups
   └─ Test restore quarterly

4. Access Control
   ├─ Application account: SELECT + INSERT + UPDATE only
   ├─ DBA account: Full admin (for integrity fixes)
   └─ Audit all data modifications (Extended Events)
```

---

### Question 7: "We're evaluating migration from log shipping to Always On AG. What are the migration steps, and what gotchas should we watch for?"

**Expected Answer (Senior DBA - Migration Engineering):**

---

**Current State vs. Target:**

```
CURRENT: Log Shipping
├─ Primary: PROD-DB-01 (OLTP)
├─ Standby: PROD-DB-02 (restore mode)
├─ Activation: Manual (takes 15 min)
├─ Failback: Manual (complex)
└─ Data sync: Every 15 minutes (log backup interval)

TARGET: Always On AG
├─ Primary: PROD-AG-01 (OLTP)
├─ Secondary: PROD-AG-02 (readable)
├─ Tertiary: PROD-AG-03 (optional, for maintenance)
├─ Failover: Automatic (< 5 sec for sync)
├─ Failback: Automatic (seamless)
└─ Data sync: Continuous (transaction-based)
```

**Migration Path:**

```
Phase 1: Prerequisites (Week 1)
  ├─ Set up Windows Failover Cluster
  ├─ Install SQL Server on both nodes
  ├─ Verify network connectivity (< 10 ms latency)
  ├─ Enable AlwaysOn on both instances
  └─ Create backup of database (required for AG seeding)

Phase 2: Build AG Infrastructure (Week 2)
  ├─ Create availability group (not yet in production)
  ├─ Configure mirrors and replicas
  ├─ Add listener
  ├─ Test automatic failover in staging environment
  └─ Verify application connection routing

Phase 3: Cutover (Weekend, Maintenance Window)
  ├─ Stop log shipping (disable jobs)
  ├─ Take final log backup on primary (log shipping)
  ├─ Restore final log on standby (log shipping)
  ├─ Promote log shipping standby to primary (standby is now online)
  ├─ Switch application connection to AG listener
  ├─ Monitor for issues (first 24 hours)
  └─ Keep log shipping standby as passive backup (if needed)

Phase 4: Cleanup (Week 3)
  ├─ Remove log shipping jobs (after AG stability)
  ├─ Archive old log shipping backups
  ├─ Update disaster recovery runbook
  └─ Document AG configuration
```

**Step-by-Step Migration During Cutover Window:**

```sql
-- FRIDAY 10 PM: Begin Migration

-- Step 1: Disable log shipping jobs on primary
EXEC msdb.dbo.sp_update_log_shipping_primary_database
    @database = 'ProductionDB',
    @ignoreremotemonitor = 1;

-- Step 2: Disable copy/restore jobs on standby
EXEC msdb.dbo.sp_update_log_shipping_secondary_database
    @secondary_database = 'ProductionDB',
    @ignoreremotemonitor = 1,
    @restore_mode = 0;

-- Step 3: Take final backup (captures last transaction)
BACKUP LOG ProductionDB 
TO DISK = '\\BackupServer\ProductionDB_final_20240328_22_00.trn'
WITH INIT;

-- Step 4: Restore final log on standby  (log shipping server)
RESTORE LOG ProductionDB
FROM DISK = '\\BackupServer\ProductionDB_final_20240328_22_00.trn'
WITH RECOVERY;  -- Brings DB online

-- Step 5: Verify data on old standby (now online)
SELECT COUNT(*) AS FinalRecordCount FROM ProductionDB.dbo.Orders;
-- Expected: All data from Friday 10 PM transaction

-- Step 6: Bring new AG online (parallel to old standby)
-- On AG Primary (PROD-AG-01):
ALTER AVAILABILITY GROUP ProductionAG ONLINE;

-- Step 7: Switch application connection string
-- OLD: Server=PROD-DB-01;
// NEW: Server=ProductionAG-Listener;

-- Application connection targets: 
//   Listener → Routes to PROD-AG-01 (primary)
//   PROD-AG-02 (secondary, readable if ApplicationIntent=ReadOnly)

-- Step 8: Verify AG is healthy
SELECT ar.replica_server_name, ars.role_desc, ars.synchronization_state_desc
FROM sys.availability_replicas ar
INNER JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id;

-- Expected:
-- PROD-AG-01: PRIMARY, SYNCHRONIZED
// PROD-AG-02: SECONDARY, SYNCHRONIZED

// Step 9: Monitor for 1 hour
// Watch for:
// - Connection errors in application logs
// - Query performance regressions
// - Replication lag
//  - Lock escalation issues
```

**Critical Gotchas & Workarounds:**

```
Gotcha 1: Application Connection Timeout During Failover
├─ Problem: Listener fails over in 5 seconds,
│           but client doesn't retry for 3 minutes (default)
├─ Impact: User sees "Connection lost" for 3 minutes
├─ Fix: Reduce client timeout; add retry logic
│   Connection string: "Connect Timeout=10;MultipleActiveResultSets=true;
│                      Connection Lifetime=300;"
└─ Prevention: Load test failover behavior before migration

Gotcha 2: Read-Only Secondary Becomes Writable on Failover
├─ Problem: Application has read-only and read-write portions
│           Both default to AG listener
├─ Impact: Read-only queries might accidentally hit primary (writes)
├─ Fix: Use separate listeners or connection intent
│   Read queries: "ApplicationIntent=ReadOnly"
│   Write queries: "ApplicationIntent=ReadWrite"
└─ Prevention: Test all app features against AG, not just primary

Gotcha 3: Distributed Transactions Unsupported Across AG
├─ Problem: Linked server queries + AG = unsupported MSDTC
├─ Impact: Batch jobs using linked servers fail
├─ Fix: Rewrite to avoid distributed transactions
│   Use: Snapshot isolation, local transactions only
└─ Prevention: Audit code before migration; identify linked servers

Gotcha 4: Backup Strategy Changes
├─ Problem: To backup from secondary (offload primary)
│           But secondary never catches up if backed up (redo lag)
├─ Impact: Backup completes, but secondary behind
├─ Fix: AUTOMATED_BACKUP_PREFERENCE = SECONDARY_ONLY
│       Backup from secondary, others act as standby
└─ Prevention: Design backup jobs before vs. after migration

Gotcha 5: SQL Agent Jobs on Secondary
├─ Problem: SQL Agent jobs run on secondary too
│           (Can execute since database is readable)
├─ Impact: Duplicate job execution; data corruption possible
├─ Fix: Disable SQL Agent on non-primary replicas
│       OR wrap jobs with "Is_Primary_Replica check"
└─ Prevention: Enable jobs only on primary replica

--Workaround for Gotcha 5: Only run jobs on primary
IF sys.fn_hadr_is_primary_replica('ProductionDB') = 1
BEGIN
    -- Run index maintenance job
    ALTER INDEX idx_Orders ON dbo.Orders REBUILD;
    UPDATE STATISTICS dbo.Orders;
END
```

**Rollback Plan (If Migration Fails):**

```sql
-- If AG has critical issues during first 24 hours:
-- Option: Failback to log shipping

-- Step 1: Failover AG back to old log shipping primary
ALTER AVAILABILITY GROUP ProductionAG FAILOVER;

-- Step 2: Application reconnects to old primary
// Connection: Server=PROD-DB-01;  (revert change)

-- Step 3: Re-enable log shipping (standby as backup)
EXEC msdb.dbo.sp_add_log_shipping_primary_database
    @database = 'ProductionDB',
    @backup_destination = '\\BackupServer\LogShipping',
    @backup_share = '\\BackupServer\LogShipping';

// Reassess AG issues, retry migration after fixes
```

**Post-Migration Validation Checklist:**

```
✓ Application connectivity working (no timeouts)
✓ Read performance unchanged
✓ Write performance unchanged
✓ Automatic failover tested (kill primary, watch failover)
✓ Failover < 30 seconds end-to-end
✓ Secondary available for reads
✓ Backup jobs updated (backup from secondary)
✓ Monitoring alerts configured (AG sync lag, replica health)
✓ Runbook updated with AG procedures (not log shipping)
✓ Team trained on AG troubleshooting
```

---

### Question 8: "Your company's SLA requires 99.95% uptime (21.6 hours downtime/year). Quarterly maintenance window cannot exceed 4 hours. Design the operational model that supports this."

**Expected Answer (Senior DBA - Operational Excellence):**

---

**SLA Math:**

```
99.95% = 21.6 hours downtime allowed/year
  ├─ Daily: 4.32 minutes downtime allowed
  ├─ Quarterly: 5.4 hours downtime allowed
  └─ Maintenance window limitation: 4 hours max
     └─ PROBLEM: Need 1.4 hours cushion for failures

Challenge: How to perform maintenance (REBUILD, UPGRADE, RESTORE tests)
          within 4-hour window while maintaining 99.95% SLA?
```

**Operational Model (Maintenance Without Downtime):**

```
Strategy: Leverage Always On to perform maintenance on SECONDARY
         without primary impact

Quarterly Maintenance Schedule:
├─ Primary (PROD-AG-01): 3 remaining "free" maintenance windows
├─ Secondary-1 (PROD-AG-02): 3 maintenance windows (rotating)
├─ Secondary-2 (PROD-AG-03): 3 maintenance windows (rotating)
│
└─ By staggering across replicas, primary always operational

Example Timeline (Q1):
  ├─ Week 1: No maintenance (failover readiness period)
  ├─ Week 2: Maintenance on SECONDARY-1 (4 hours)
  │  │       ├─ Index rebuild offline
  │  │       └─ Update cumulative hotfix
  │  │       Primary: ONLINE, serving users
  │  │
  │  └─ Catch-up: SECONDARY-1 resynchronizes with primary
  │
  ├─ Week 3: Maintenance on SECONDARY-2 (4 hours)
  │  └─ Same maintenance as Secondary-1
  │
  ├─ Week 12 (end of Q1): Maintenance on PRIMARY (must be coordinated)
     └─ Failover to SECONDARY-1
     └─ Perform maintenance on primary offline
     └─ Failback when complete
```

**Implementation Details:**

```sql
-- Quarterly Maintenance Procedure on Secondary
-- (Does not affect primary availability)

-- 1. Primary still serving all user transactions
SELECT COUNT(*) FROM sys.dm_exec_requests
WHERE session_id > 50 AND status = 'Running';  -- Thousands of queries running

-- 2. On SECONDARY replica, disable replication temporarily
ALTER AVAILABILITY GROUP ProductionAG
MODIFY REPLICA ON 'PROD-AG-02'
WITH (AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT);
-- Secondary can fall behind primary without blocking primary

-- 3. Run full index maintenance (4-hour window)
-- REORGANIZE for 10-20% fragmentation
-- REBUILD for > 20% fragmentation

-- 3a. Identify fragmented indexes
SELECT TOP 10
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- 3b. REBUILD indexes (ONLINE if available)
ALTER INDEX idx_Orders_PK ON dbo.Orders 
REBUILD WITH (ONLINE = ON, FILLFACTOR = 80);
-- Takes ~30 min per index, completion time: 2-3 hours

-- 3c. Update statistics (full rebuild)
UPDATE STATISTICS dbo.Orders;
UPDATE STATISTICS dbo.Customers;
-- Takes ~20 minutes

-- 4. Check for corruption (DBCC CHECKDB)
DBCC CHECKDB (ProductionDB, REPAIR_REBUILD);
-- Takes ~1 hour for 2 TB database

-- 5. Re-enable replication on secondary
ALTER AVAILABILITY GROUP ProductionAG
MODIFY REPLICA ON 'PROD-AG-02'
WITH (AVAILABILITY_MODE = SYNCHRONOUS_COMMIT);

-- 6. Wait for secondary to catch up with primary
-- Monitor redo queue until zero
WHILE (SELECT redo_queue_size FROM sys.dm_hadr_database_replica_states
       WHERE database_id = DB_ID('ProductionDB')) > 0
BEGIN
    WAITFOR DELAY '00:00:05';
    SELECT 'Secondary catching up...' AS Status;
END
```

**Maintenance Without Primary Impact:**

```
Benefits:
├─ Users see zero maintenance downtime
├─ All queries continue against primary
├─ Each replica gets full maintenance quarterly
├─ No rush to complete maintenance (4 hours available)
├─ Can schedule maintenance on rotation
└─ SLA maintained: 99.95% = ✓

Timeline Example (All Quarterly Maintenance)

Q1 Schedule:
├─ Week 2 (Monday-Tuesday, 2 AM-6 AM):
│  └─ Secondary-1 maintenance
│     ├─ Primary: ONLINE, serving 100% traffic
│     ├─ Secondary-1: MAINTENANCE (full rebuild + CHECKDB)
│     └─ Secondary-2: Standby (async, safe)
│
├─ Week 6 (Monday-Tuesday, 2 AM-6 AM):
│  └─ Secondary-2 maintenance
│
└─ Week 12 (Monday-Tuesday, 2 AM-6 AM):
   └─ Primary maintenance (requires failover)
      ├─ Failover to Secondary-1 (<5 seconds)
      ├─ Primary: Now offline for maintenance
      ├─ Secondary-1: New primary (100% traffic)
      └─ After maintenance: Failback to original primary
```

**Quarterly System Patching (4-Hour Window):**

```
Schedule: Q1 = Jan/Feb/Mar, Q2 = Apr/May/Jun, etc.

Windows patching schedule (Monday 2 AM UTC):
├─ Patch Secondary-1: Month 1 of quarter
├─ Patch Secondary-2: Month 2 of quarter
└─ Patch Primary: Month 3 of quarter (requires failover)

Each patching cycle:
├─ Apply OS patches (30 min: install, don't restart)
├─ Apply SQL Server CUs (30 min: apply, don't restart)
├─ Restart instance (15 min: SQL Server comes back online)
├─ Resynchronize replica (30-60 min: catch up with primary)
├─ Validate health (15 min: confirm no issues)
└─ Total: ~3 hours (within 4-hour window)
```

**Uptime Tracking & SLA Monitoring:**

```sql
-- Track actual uptime (monthly report)
CREATE TABLE UptimeTracker (
    MonthYear DATE,
    PlannedDowntime NUMERIC(10,2),  -- Maintenance window
    UnplannedDowntime NUMERIC(10,2),  -- Failover, issues
    TotalDowntime NUMERIC(10,2),
    AvailabilityPercent NUMERIC(5,2)
);

-- Example monthly data:
INSERT INTO UptimeTracker VALUES
('2024-01-31', 0.0, 0.02, 0.02, 99.973),  -- 2 min unplanned downtime
('2024-02-29', 0.0, 0.01, 0.01, 99.983),
('2024-03-31', 4.0, 0.05, 4.05, 99.722);  -- 4 hours maintenance

-- Q1 Aggregate
SELECT
    'Q1 2024' AS Quarter,
    SUM(UnplannedDowntime) AS TotalUnplannedDowntime,
    CAST(100 - (SUM(UnplannedDowntime) * 100 / (24*7*13)) AS NUMERIC(5,2)) AS Q1_Availability
FROM UptimeTracker
WHERE MONTH(MonthYear) IN (1, 2, 3);
-- Result: Q1 Availability = 99.957% (meets 99.95% SLA!)
```

**Cost/Resource Implications:**

```
Requirements for 99.95% SLA with 4-hour maintenance window:
├─ 3 SQL Server instances (primary + 2 secondaries)
├─ Windows Failover Cluster (licensing)
├─ Always On Availability Groups (Enterprise Edition required)
├─ Monitoring & alerting infrastructure
└─ 24/7 on-call DBA rotation

Cost Estimate:
├─ Hardware: $300K (3 high-performance servers)
├─ SQL Server licenses: $150K (Enterprise Edition × 3)
├─ Infrastructure (network, storage): $200K/year
├─ Operations (DBA salaries, monitoring): $400K/year
└─ Total annual: ~$600K/year

ROI: Downtime costs $X per minute in lost revenue
      → 99.95% SLA reduces financial risk significantly
```

---

This comprehensive study guide is now **complete** with all core sections, practical code examples, real-world scenarios, and technical interview questions suitable for senior database administrators. The guide covers enterprise-scale SQL Server HA/DR, security, maintenance, and monitoring strategies grounded in production experience and compliance requirements.

---


