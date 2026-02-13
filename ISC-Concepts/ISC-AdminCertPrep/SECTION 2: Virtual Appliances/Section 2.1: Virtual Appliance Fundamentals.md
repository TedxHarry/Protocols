# Section 2.1: Virtual Appliance Fundamentals

**Master the core concepts of Virtual Appliances, their role in ISC, and architecture essentials for the certification exam**

---

## Table of Contents

- [2.1.1 What are Virtual Appliances?](#211-what-are-virtual-appliances)
- [2.1.2 Role of VAs in Identity Security Cloud](#212-role-of-vas-in-identity-security-cloud)
- [2.1.3 VA Architecture and Components](#213-va-architecture-and-components)

---

# 2.1.1 What are Virtual Appliances?

**Learning Objective:** Understand what Virtual Appliances are and when they're needed.

---

## Definition and Purpose

**Virtual Appliance (VA)** = Virtual machine that bridges SailPoint ISC (cloud) to on-premises systems

**Key Concept:**
```
ISC (Cloud) ←→ VA (Your Network) ←→ On-Premises Sources (AD, Databases)
```

**Why VAs exist:**
- ISC is in the cloud, can't reach on-premises systems directly
- VA provides secure connection without opening firewall inbound ports
- VA sits inside your network, accesses local resources
- VA connects outbound to ISC (security-approved)

---

## When You Need a VA

### **Need VA:**
✅ Active Directory  
✅ LDAP  
✅ On-premises databases (Oracle, SQL Server, MySQL)  
✅ File-based sources (CSV on network share)  
✅ Any system behind corporate firewall  

### **Don't Need VA:**
❌ Salesforce  
❌ Workday  
❌ Office 365 / Azure AD  
❌ ServiceNow  
❌ Any cloud/SaaS application  

**Rule:** If it's on-premises, you need a VA. If it's cloud/SaaS, ISC connects directly.

---

## How VAs Work

**Communication Flow:**
```
1. VA initiates outbound HTTPS (port 443) to ISC
2. ISC sends commands to VA (via existing connection)
3. VA connects to on-prem source (local network)
4. VA executes operation (read/write data)
5. VA sends results back to ISC
```

**Critical Security Point:**
- VA only makes OUTBOUND connections to ISC
- No inbound firewall rules required
- ISC never initiates connections to VA
- All communication over encrypted HTTPS

---

## VA Deployment Types

| Type | Configuration | Use Case |
|------|--------------|----------|
| **Single VA** | 1 VA | Dev/test, small environments (<5K users) |
| **Clustered (HA)** | 2+ VAs, failover | Production, high availability required |
| **Distributed** | VAs in multiple locations | Global deployment, regional sources |

**Exam Tip:** Know that production environments should use clustered VAs for redundancy.

---

## Quick Reference

**VA Essentials:**
- **What:** VM bridging cloud ISC to on-prem systems
- **Why:** Secure access without firewall changes
- **When:** On-premises sources only
- **How:** Outbound HTTPS connection to ISC
- **Port:** 443 (HTTPS) outbound only

**Common Misconception:** VAs don't store identity data—they're a pass-through bridge.

---

## Practice Exercise

**Question:** You have Azure AD, Active Directory, and Salesforce. Which need VAs?

<details>
<summary>Answer</summary>

**Active Directory ONLY**

- ✅ Active Directory = On-premises → Needs VA
- ❌ Azure AD = Cloud → Direct SaaS connector
- ❌ Salesforce = Cloud → Direct SaaS connector

You need: **1 Virtual Appliance** (for AD only)
</details>

---

# 2.1.2 Role of VAs in Identity Security Cloud

**Learning Objective:** Understand what VAs do in the ISC architecture.

---

## Four Primary Roles

### **1. Source Connector**
- Authenticates to on-premises sources
- Executes queries (LDAP, SQL, file reads)
- Manages connection pooling

### **2. Data Aggregator**
- Reads identity/account data from sources
- Sends data to ISC for processing
- Handles delta detection (changed records only)

### **3. Provisioning Engine**
- Creates accounts (AD, databases)
- Modifies accounts (attributes, groups)
- Deletes/disables accounts
- Executes password resets

### **4. Password Manager**
- Resets passwords in AD
- Validates password policies
- Synchronizes passwords (if configured)

---

## VA in Common Workflows

### **New Hire (Joiner):**
```
ISC creates identity → VA creates AD account → VA adds to groups → Done
```

### **Termination (Leaver):**
```
ISC detects inactive → VA disables AD account → VA removes access → Done
```

### **Access Request:**
```
User requests VPN → Approved in ISC → VA adds user to AD group → Access granted
```

**Key Point:** ISC makes decisions, VA executes actions on-premises.

---

## Data Flow Examples

**Aggregation (Read):**
```
ISC schedules aggregation → VA queries AD → VA sends data to ISC → ISC updates identities
```

**Provisioning (Write):**
```
ISC sends create command → VA creates AD account → VA reports success → ISC updates record
```

---

## VA vs Direct Connector

| Feature | VA (On-Prem) | Direct (SaaS) |
|---------|--------------|---------------|
| **Location** | Your network | ISC cloud |
| **Firewall** | Outbound only | Direct internet |
| **Examples** | AD, LDAP, databases | Salesforce, Workday |
| **Maintenance** | Customer manages VM | SailPoint manages |
| **Latency** | Local network speed | Internet dependent |

---

## High Availability (HA)

**Clustered VAs:**
- 2+ VAs working together
- Automatic failover if one fails
- Load balancing across VAs
- Production best practice

**Behavior:**
```
VA-01 fails → ISC detects → Routes to VA-02 → No downtime
```

**Exam Tip:** Production environments should use VA clustering for HA.

---

## Practice Exercise

**Question:** What happens if a VA goes offline during account aggregation?

<details>
<summary>Answer</summary>

**Without Clustering:**
- Aggregation fails
- ISC marks operation as failed
- Queued provisioning commands wait in ISC
- When VA reconnects, ISC resends commands
- Aggregation can be retried

**With Clustering (HA):**
- ISC immediately fails over to VA-02
- Aggregation continues on secondary VA
- No service interruption
- Transparent to users

**Best Practice:** Use clustered VAs in production to prevent downtime.
</details>

---

# 2.1.3 VA Architecture and Components

**Learning Objective:** Understand VA internal components and technical requirements.

---

## Core Components

### **1. VA Management Service**
- Main control process
- Manages VA lifecycle (startup/shutdown)
- Configuration management
- Health monitoring

### **2. Cloud Connector Gateway (CCG)**
- Maintains connection to ISC
- Sends heartbeat every 60 seconds
- Receives commands from ISC
- Sends results back to ISC
- Handles reconnection if connection lost

### **3. Connector Framework**
- Loads specific connectors (AD, JDBC, LDAP)
- Manages connector lifecycle
- Handles schema discovery
- Executes operations

### **4. Individual Connectors**
- **AD Connector:** LDAP/LDAPS to domain controllers
- **JDBC Connector:** Database connections via JDBC
- **LDAP Connector:** Generic LDAP directories
- **File Connector:** CSV/XML files

---

## System Architecture
```
┌────────────────────────────────┐
│      SailPoint ISC (Cloud)     │
└────────────┬───────────────────┘
             │ HTTPS (443)
             │ Outbound from VA
             ↓
┌────────────────────────────────┐
│    Virtual Appliance (VM)      │
│                                │
│  ┌──────────────────────────┐ │
│  │  VA Management + CCG     │ │
│  └──────────────────────────┘ │
│                                │
│  ┌────────┐ ┌────────┐       │
│  │   AD   │ │  JDBC  │       │
│  │Connector│ │Connector│      │
│  └────────┘ └────────┘       │
└─────┬──────────────┬──────────┘
      │              │
      ↓              ↓
┌──────────┐  ┌──────────┐
│ Active   │  │    HR    │
│Directory │  │ Database │
└──────────┘  └──────────┘
```

---

## Resource Requirements

**Minimum Specs:**

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **OS** | Linux 64-bit | RHEL/CentOS/Ubuntu |
| **CPU** | 4 vCPUs | 8 vCPUs |
| **RAM** | 8 GB | 16 GB |
| **Disk** | 100 GB | 200 GB |
| **Network** | 100 Mbps | 1 Gbps |

**Sizing Guide:**

| Identities | CPU | RAM | Deployment |
|------------|-----|-----|------------|
| **<5K** | 4 | 8 GB | Single VA |
| **5K-25K** | 8 | 16 GB | Single or HA |
| **25K-100K** | 16 | 32 GB | HA required |
| **100K+** | 16+ | 32+ GB | Multiple VAs |

---

## Key File Locations
```
/opt/sailpoint/
├── config/           (VA configuration)
├── logs/
│   ├── va.log       (VA service)
│   ├── connector.log (Connector operations)
│   └── ccg.log      (Gateway communications)
├── cache/           (Temporary data)
└── bin/             (Executables)
```

**Exam Tip:** Know that connector logs are in `/opt/sailpoint/logs/connector.log`

---

## Performance Characteristics

**Aggregation Throughput:**
- Active Directory: 1,000-2,000 users/minute
- Database: 500-1,500 records/minute
- Depends on: attributes, network latency, VA resources

**Provisioning Speed:**
- Account creation: 2-5 seconds
- Account modification: 1-2 seconds
- Concurrent operations: 5-10 in parallel

---

## Monitoring and Health

**Healthy VA Indicators:**
- ✅ CPU < 60%
- ✅ Memory < 70%
- ✅ Disk > 20% free
- ✅ Connected to ISC
- ✅ Heartbeat every 60 seconds

**Unhealthy Indicators:**
- ❌ CPU > 80%
- ❌ Memory > 85%
- ❌ Missed heartbeats (>3 minutes)
- ❌ Failed operations

**Check VA Status:**
- In ISC: Admin → Virtual Appliances
- On VA: `/opt/sailpoint/logs/`

---

## Security Components

**Certificate Management:**
- VA uses client certificate to authenticate to ISC
- Auto-renewed by ISC
- Valid for 1 year

**Credential Storage:**
- Source passwords stored encrypted (AES-256)
- Local encrypted keystore on VA
- Never transmitted in plaintext

**Communication:**
- All VA ↔ ISC traffic encrypted (TLS 1.2+)
- Port 443 HTTPS only
- Certificate validation enforced

---

## Practice Exercises

**Exercise 1:** A VA is at 90% CPU during aggregation. What should you check?

<details>
<summary>Answer</summary>

**Check:**
1. **VA sizing:** Is 4 CPU enough for 50K users? (Probably not)
2. **Concurrent operations:** Multiple aggregations running?
3. **Schedule optimization:** Stagger aggregation times
4. **Query efficiency:** Optimize LDAP filters, reduce attributes

**Solutions:**
- Increase CPU/RAM resources
- Schedule aggregations during off-peak hours
- Use delta aggregation instead of full
- Consider adding additional VA for load distribution
</details>

---

**Exercise 2:** Where would you find logs for a failed AD aggregation?

<details>
<summary>Answer</summary>

**Primary location:**
```bash
/opt/sailpoint/logs/connector.log
```

**Search for errors:**
```bash
grep -i "error" /opt/sailpoint/logs/connector.log
grep "Active Directory" /opt/sailpoint/logs/connector.log
```

**Other relevant logs:**
- `/opt/sailpoint/logs/va.log` - VA service events
- `/opt/sailpoint/logs/ccg.log` - ISC communication

**In ISC UI:**
- Admin → Virtual Appliances → View logs
- Activity → Provisioning → Filter by failed
</details>

---

**Exercise 3:** What happens to queued provisioning if VA reboots?

<details>
<summary>Answer</summary>

**Commands Queued in ISC:**
- ✅ Preserved in ISC
- ✅ NOT lost during VA reboot
- ✅ Execute when VA reconnects

**In-Progress Operations:**
- ❌ Interrupted/failed
- ❌ ISC marks as failed
- ❌ May need manual verification

**When VA Restarts:**
1. VA reconnects to ISC
2. ISC sends queued commands
3. Operations execute normally

**With VA Clustering:**
- VA-01 reboots → VA-02 takes over
- No interruption, no queued operations
- This is why HA matters!
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **What VAs Are:**
- Bridge between ISC (cloud) and on-premises systems
- Outbound connection only (port 443 HTTPS)
- Required for on-prem sources, not for cloud/SaaS

✅ **What VAs Do:**
- Connect to sources (AD, LDAP, databases)
- Aggregate data (read)
- Provision accounts (write)
- Manage passwords

✅ **VA Architecture:**
- CCG manages ISC connection (heartbeat every 60s)
- Connectors handle source-specific operations
- Logs in `/opt/sailpoint/logs/`
- Minimum: 4 CPU, 8 GB RAM, 100 GB disk

✅ **Production Best Practices:**
- Use clustered VAs for HA
- Monitor health (CPU, memory, disk, heartbeat)
- Size appropriately for identity count
- Schedule aggregations during off-peak hours

**Exam Focus Areas:**
- When VA is needed vs. direct SaaS connector
- VA roles in joiner/mover/leaver workflows
- VA clustering for high availability
- Basic troubleshooting (logs, health metrics)
- Resource requirements and sizing

---

**End of Section 2.1: Virtual Appliance Fundamentals**

**Next:** Section 2.2: VA Installation and Configuration

---

**Study Tips:**
- Focus on when VA is required (on-prem) vs. not (cloud/SaaS)
- Understand VA's role in workflows (executes, doesn't decide)
- Know basic troubleshooting: logs location, health indicators
- Remember: Production = clustered VAs for HA
