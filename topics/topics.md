# Database Administration Learning Topics

---

## MS SQL Server

### Phase-1.0: SQL Server Architecture
- Topic: SQL Server Architecture
- Sub-Topics: SQL Server service architecture, Database engine vs SQL OS, SQLOS schedulers, Worker threads, Memory clerks, Lazy writer, Checkpoint process, Buffer pool, Plan cache

### Phase-2.0: Storage & Files
- Topic: Storage & Files
- Sub-Topics: MDF vs LDF, Data file internals, Log file internals, VLFs, Autogrowth strategy, Instant File Initialization, tempdb file layout

### Phase-3.0: Memory Management
- Topic: Memory Management
- Sub-Topics: Max server memory, Min server memory, Buffer pool extension, Memory pressure detection, NUMA awareness

### Phase-4.0: CPU & Concurrency
- Topic: CPU & Concurrency
- Sub-Topics: SOS scheduler, Runnable vs waiting tasks, CXPACKET, CXCONSUMER, MAXDOP, Cost threshold for parallelism

### Phase-5.0: Transactions & Logging
- Topic: Transactions & Logging
- Sub-Topics: ACID, Write-ahead logging, Checkpoints, Log truncation, Log reuse wait types

### Phase-6.0: Index Internals
- Topic: Index Internals
- Sub-Topics: Heap vs clustered index, B-Tree structure, Non-clustered indexes, Included columns, Filtered indexes, Index fragmentation

### Phase-7.0: Statistics
- Topic: Statistics
- Sub-Topics: Auto create statistics, Auto update statistics, Sampling, Cardinality estimation, Async stats update

### Phase-8.0: Query Processing
- Topic: Query Processing
- Sub-Topics: Query optimizer, Algebrizer, Execution plan types, Estimated vs actual plans, Parameter sniffing, Plan cache pollution

### Phase-9.0: Performance Tuning
- Topic: Performance Tuning
- Sub-Topics: Wait statistics, IO waits, CPU waits, Lock waits, Latch waits

### Phase-10.0: tempdb Mastery
- Topic: tempdb Mastery
- Sub-Topics: tempdb contention, GAM / SGAM, Version store, tempdb spills

### Phase-11.0: Locking & Blocking
- Topic: Locking & Blocking
- Sub-Topics: Lock modes, Lock escalation, Deadlocks, Isolation levels, Snapshot isolation

### Phase-12.0: Isolation & Concurrency
- Topic: Isolation & Concurrency
- Sub-Topics: Read committed snapshot, Serializable, Phantom reads

### Phase-13.0: Execution Plan Operators
- Topic: Execution Plan Operators
- Sub-Topics: Nested loops, Hash match, Merge join, Index seek, Index scan

### Phase-14.0: Query Store
- Topic: Query Store
- Sub-Topics: Plan forcing, Regressions, Query baselines

### Phase-15.0: Backup Internals
- Topic: Backup Internals
- Sub-Topics: Full backup, Differential backup, Log backup, Copy-only backup

### Phase-16.0: Restore & Recovery
- Topic: Restore & Recovery
- Sub-Topics: Restore sequence, Point-in-time recovery, Page-level restore, Tail-log backup

### Phase-17.0: HA/DR Fundamentals
- Topic: HA/DR Fundamentals
- Sub-Topics: RPO vs RTO, Failover concepts

### Phase-18.0: Always On Availability Groups
- Topic: Always On Availability Groups
- Sub-Topics: Sync vs async, Read replicas, Listener, Quorum

### Phase-19.0: Replication
- Topic: Replication
- Sub-Topics: Snapshot replication, Transactional replication, Merge replication

### Phase-20.0: Log Shipping
- Topic: Log Shipping
- Sub-Topics: Architecture, Monitoring

### Phase-21.0: Security Basics
- Topic: Security Basics
- Sub-Topics: Authentication, Authorization, Server roles, Database roles

### Phase-22.0: Advanced Security
- Topic: Advanced Security
- Sub-Topics: TDE, Always Encrypted, Auditing, Row-level security

### Phase-23.0: Maintenance
- Topic: Maintenance
- Sub-Topics: Index maintenance, Statistics maintenance, Integrity checks

### Phase-24.0: Monitoring & Troubleshooting
- Topic: Monitoring & Troubleshooting
- Sub-Topics: DMVs, Extended Events, SQL Server logs

### Phase-25.0: Automation
- Topic: Automation
- Sub-Topics: SQL Agent jobs, PowerShell, Maintenance plans

### Phase-26.0: Cloud SQL Server
- Topic: Cloud SQL Server
- Sub-Topics: SQL Server on EC2, RDS SQL Server, Licensing

### Phase-27.0: Capacity Planning
- Topic: Capacity Planning
- Sub-Topics: Growth forecasting, IO sizing, CPU sizing

### Phase-28.0: Patching & Upgrades
- Topic: Patching & Upgrades
- Sub-Topics: CU strategy, In-place upgrade, Side-by-side upgrade

### Phase-29.0: Disaster Scenarios
- Topic: Disaster Scenarios
- Sub-Topics: Corruption, Disk failure, Human error

### Phase-30.0: Enterprise Operations
- Topic: Enterprise Operations
- Sub-Topics: Change management, Incident response, Root cause analysis

---

## SSRS (SQL Server Reporting Services)

### Phase-1.0: SSRS Fundamentals
- Topic: SSRS Fundamentals
- Sub-Topics: What SSRS is, SSRS architecture, Report Server vs Report Manager

### Phase-2.0: SSRS Installation
- Topic: SSRS Installation
- Sub-Topics: Native mode, Report Server configuration

### Phase-3.0: Data Sources
- Topic: Data Sources
- Sub-Topics: Shared data sources, Embedded data sources

### Phase-4.0: Datasets
- Topic: Datasets
- Sub-Topics: Query-based datasets, Stored procedure datasets

### Phase-5.0: Report Design Basics
- Topic: Report Design Basics
- Sub-Topics: Tables, Matrices, Lists

### Phase-6.0: Parameters
- Topic: Parameters
- Sub-Topics: Report parameters, Cascading parameters

### Phase-7.0: Expressions
- Topic: Expressions
- Sub-Topics: SSRS expression language, Conditional formatting

### Phase-8.0: Grouping & Aggregates
- Topic: Grouping & Aggregates
- Sub-Topics: Row groups, Column groups

### Phase-9.0: Charts & Visuals
- Topic: Charts & Visuals
- Sub-Topics: Bar charts, Line charts

### Phase-10.0: Subreports
- Topic: Subreports
- Sub-Topics: Subreport architecture, Parameter passing

### Phase-11.0: Drill-Down & Drill-Through
- Topic: Drill-Down & Drill-Through
- Sub-Topics: Interactive reports, Navigation

### Phase-12.0: Security
- Topic: Security
- Sub-Topics: Role-based security, Folder permissions

### Phase-13.0: Performance
- Topic: Performance
- Sub-Topics: Dataset caching, Report snapshots

### Phase-14.0: Subscriptions
- Topic: Subscriptions
- Sub-Topics: Standard subscriptions, Data-driven subscriptions

### Phase-15.0: Deployment
- Topic: Deployment
- Sub-Topics: Visual Studio deployment, Folder structure

### Phase-16.0: Maintenance
- Topic: Maintenance
- Sub-Topics: Report backups, Migration

### Phase-17.0: Troubleshooting
- Topic: Troubleshooting
- Sub-Topics: Execution logs, Report timeouts

---

## SSIS (SQL Server Integration Services)

### Phase-1.0: SSIS Fundamentals
- Topic: SSIS Fundamentals
- Sub-Topics: What SSIS is, ETL vs ELT, SSIS architecture

### Phase-2.0: SSIS Components
- Topic: SSIS Components
- Sub-Topics: Control flow, Data flow

### Phase-3.0: SSIS Project Structure
- Topic: SSIS Project Structure
- Sub-Topics: Projects, Packages

### Phase-4.0: Connection Managers
- Topic: Connection Managers
- Sub-Topics: OLE DB, ADO.NET, Flat file

### Phase-5.0: Control Flow Tasks
- Topic: Control Flow Tasks
- Sub-Topics: Execute SQL task, Data Flow task

### Phase-6.0: Precedence Constraints
- Topic: Precedence Constraints
- Sub-Topics: Success, Failure

### Phase-7.0: Data Flow Basics
- Topic: Data Flow Basics
- Sub-Topics: Source components, Destination components

### Phase-8.0: Transformations
- Topic: Transformations
- Sub-Topics: Derived column, Lookup, Conditional split

### Phase-9.0: Advanced Transformations
- Topic: Advanced Transformations
- Sub-Topics: Merge, Merge join, Fuzzy lookup

### Phase-10.0: Error Handling
- Topic: Error Handling
- Sub-Topics: Error outputs, Redirect rows

### Phase-11.0: Variables & Parameters
- Topic: Variables & Parameters
- Sub-Topics: Package variables, Project parameters

### Phase-12.0: Expressions
- Topic: Expressions
- Sub-Topics: SSIS expression language

### Phase-13.0: Logging
- Topic: Logging
- Sub-Topics: SSIS logging, Event handlers

### Phase-14.0: Performance Tuning
- Topic: Performance Tuning
- Sub-Topics: Buffer size, Parallelism

### Phase-15.0: Transactions
- Topic: Transactions
- Sub-Topics: Transaction support, Isolation levels

### Phase-16.0: Deployment Model
- Topic: Deployment Model
- Sub-Topics: Project deployment model, SSIS Catalog

### Phase-17.0: Security
- Topic: Security
- Sub-Topics: Sensitive data protection, Package encryption

### Phase-18.0: Scheduling
- Topic: Scheduling
- Sub-Topics: SQL Server Agent, Job configuration

### Phase-19.0: Configuration Management
- Topic: Configuration Management
- Sub-Topics: Environment variables, Parameterization

### Phase-20.0: Incremental Loads
- Topic: Incremental Loads
- Sub-Topics: Change tracking, CDC

### Phase-21.0: Troubleshooting
- Topic: Troubleshooting
- Sub-Topics: Execution reports, Failed package recovery

### Phase-22.0: Maintenance
- Topic: Maintenance
- Sub-Topics: Package versioning, Migration

### Phase-23.0: Cloud Awareness
- Topic: Cloud Awareness
- Sub-Topics: SSIS on Azure, Lift-and-shift patterns

---

*Generated from topics.json on 2026-03-23*
