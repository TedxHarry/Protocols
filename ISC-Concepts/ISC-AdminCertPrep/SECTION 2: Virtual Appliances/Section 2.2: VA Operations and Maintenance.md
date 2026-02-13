# Section 2.2: VA Operations and Maintenance

**Master monitoring VA health, reading logs, and troubleshooting common VA issues**

---

## Table of Contents

- [2.2.1 Monitoring VA Health Status](#221-monitoring-va-health-status)
- [2.2.2 Understanding VA Logs](#222-understanding-va-logs)
- [2.2.3 Basic VA Troubleshooting Techniques](#223-basic-va-troubleshooting-techniques)

---

# 2.2.1 Monitoring VA Health Status

**Learning Objective:** Monitor Virtual Appliance health and identify potential issues.

---

## VA Health Indicators

### **Primary Health Metrics**

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| **CPU Usage** | <60% | 60-80% | >80% |
| **Memory Usage** | <70% | 70-85% | >85% |
| **Disk Space** | >20% free | 10-20% free | <10% free |
| **ISC Connection** | Connected | Intermittent | Disconnected |
| **Heartbeat** | Every 60s | Delayed | Missed (>3 min) |

**Exam Tip:** Healthy VA = CPU <60%, Memory <70%, Disk >20% free, Connected to ISC.

---

## Monitoring VA in ISC

### **Check VA Status in UI**

**Navigation:**
```
Admin → Virtual Appliances
```

**What You'll See:**
```
Virtual Appliances

Name: VA-PROD-01
Status: ● AVAILABLE (Green)
Last Heartbeat: 2024-02-13 10:45:23 (1 minute ago)
Version: 8.4.2
Cluster: Production-Cluster

Name: VA-PROD-02
Status: ● AVAILABLE (Green)
Last Heartbeat: 2024-02-13 10:45:18 (1 minute ago)
Version: 8.4.2
Cluster: Production-Cluster
```

**Status Meanings:**

| Status | Color | Meaning | Action |
|--------|-------|---------|--------|
| **AVAILABLE** | Green | Healthy, connected | None needed |
| **WARNING** | Yellow | Degraded performance | Investigate |
| **UNAVAILABLE** | Red | Disconnected | Immediate action |
| **UNKNOWN** | Gray | No status data | Check VA |

### **VA Details View**

**Click on VA name to see details:**
```
VA-PROD-01 Details:

System Information:
  IP Address: 10.10.50.25
  Hostname: va-prod-01.company.com
  OS: Red Hat Enterprise Linux 8.5
  VA Version: 8.4.2
  
Health Metrics:
  CPU: 45% (Normal)
  Memory: 62% (Normal)
  Disk: 35% free (Normal)
  
Connection:
  Status: Connected
  Last Heartbeat: 45 seconds ago
  Uptime: 15 days, 4 hours
  
Sources Using This VA:
  - Active Directory (source-ad-01)
  - HR Database (source-hr-db)
  - LDAP Directory (source-ldap-01)
```

---

## Monitoring VA on the System

### **Check VA Service Status**
```bash
# Check if VA service is running
sudo systemctl status sailpoint-va

# Expected output (healthy):
● sailpoint-va.service - SailPoint Virtual Appliance
   Loaded: loaded (/etc/systemd/system/sailpoint-va.service)
   Active: active (running) since Mon 2024-01-29 08:00:00 EST; 2 weeks ago
   Main PID: 1234 (va)
   Memory: 2.5G
   CGroup: /system.slice/sailpoint-va.service
```

**Service Commands:**
```bash
# Start VA
sudo systemctl start sailpoint-va

# Stop VA (graceful shutdown)
sudo systemctl stop sailpoint-va

# Restart VA
sudo systemctl restart sailpoint-va

# Enable autostart on boot
sudo systemctl enable sailpoint-va
```

### **Check System Resources**

**CPU and Memory:**
```bash
# Quick view
top

# Look for 'va' or 'java' process
# CPU% and MEM% columns show usage

# Alternative
htop  # More user-friendly (if installed)
```

**Disk Space:**
```bash
# Check disk usage
df -h

# Look for /opt/sailpoint partition
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1       200G  130G   70G   65%   /

# Detailed directory usage
du -sh /opt/sailpoint/*
5.2G    /opt/sailpoint/cache
2.1G    /opt/sailpoint/logs
500M    /opt/sailpoint/config
```

**Network Connectivity:**
```bash
# Test connection to ISC
curl -I https://yourcompany.identitynow.com

# Expected: HTTP/1.1 200 OK or similar

# Check if VA is listening
netstat -tulpn | grep va

# Or
ss -tulpn | grep va
```

---

## Heartbeat Monitoring

### **Understanding Heartbeat**

**What is heartbeat:**
- VA sends "I'm alive" signal to ISC every 60 seconds
- Includes health metrics (CPU, memory, disk)
- ISC uses this to mark VA as AVAILABLE

**Heartbeat flow:**
```
Every 60 seconds:
  VA → ISC: Heartbeat message
    {
      "status": "healthy",
      "cpu": 45,
      "memory": 62,
      "disk": 35,
      "timestamp": "2024-02-13T10:45:23Z"
    }
  
  ISC → VA: Acknowledged
```

**When heartbeat fails:**
```
ISC doesn't receive heartbeat for 3+ minutes:
  → ISC marks VA as UNAVAILABLE
  → Queues operations for when VA returns
  → Sends alert (if configured)
  → Fails over to other VA (if clustered)
```

**Check heartbeat in logs:**
```bash
# View CCG (Cloud Gateway) logs
tail -f /opt/sailpoint/logs/ccg.log

# Look for:
[INFO] Heartbeat sent to ISC
[INFO] Heartbeat acknowledged
```

**Exam Tip:** VA heartbeat is every 60 seconds. After 3+ missed heartbeats, ISC marks VA as UNAVAILABLE.

---

## Performance Monitoring

### **Key Performance Indicators**

**Aggregation Performance:**
```
Active Directory Aggregation:
  Expected: 1,000-2,000 users/minute
  
  10,000 users should complete in: 5-10 minutes
  100,000 users should complete in: 60-90 minutes
  
Signs of Performance Issues:
  - Taking 2x+ expected time
  - High CPU (>80%) during aggregation
  - Memory climbing steadily
  - Disk I/O bottleneck
```

**Provisioning Performance:**
```
Account Operations:
  Create account: 2-5 seconds
  Modify account: 1-2 seconds
  
Signs of Issues:
  - Operations taking >30 seconds
  - Provisioning timeouts
  - Queue backing up
```

### **Monitoring During Aggregation**

**Check aggregation progress:**
```bash
# Watch connector logs in real-time
tail -f /opt/sailpoint/logs/connector.log

# You'll see:
[INFO] Aggregation started: Active Directory
[INFO] Fetched 1000 accounts...
[INFO] Fetched 2000 accounts...
[INFO] Fetched 3000 accounts...
[INFO] Aggregation completed: 10,247 accounts in 6m 23s
```

**Monitor system during aggregation:**
```bash
# Watch CPU/memory while aggregation runs
watch -n 5 'top -b -n 1 | head -20'

# Should see:
# - CPU spike during aggregation (normal)
# - Memory increase (normal if within limits)
# - Returns to baseline when complete
```

---

## Alert Configuration (ISC)

### **Setting Up VA Health Alerts**

**In ISC (if available in your tenant):**
```
Admin → Settings → Notifications

Create Alert:
  Name: VA Disconnected
  Trigger: VA status changes to UNAVAILABLE
  Recipients: iam-ops@company.com
  Delivery: Email

Create Alert:
  Name: VA High CPU
  Trigger: VA CPU > 80% for 10 minutes
  Recipients: iam-ops@company.com
  Delivery: Email
```

**Exam Note:** Alert configuration varies by tenant. Know that alerts CAN be configured for VA health issues.

---

## Clustering Health (HA Deployments)

### **Monitor Cluster Status**

**Clustered VAs in ISC:**
```
Admin → Virtual Appliances

Cluster: Production-Cluster
  VA-PROD-01: ● AVAILABLE
  VA-PROD-02: ● AVAILABLE
  
Health: ✓ All members healthy
Load Distribution: Balanced
```

**What to monitor:**
- All cluster members showing AVAILABLE
- Load balanced across members
- No single VA overloaded
- Failover working (test periodically)

**Test failover:**
```
1. Stop VA-01
   sudo systemctl stop sailpoint-va
   
2. Check ISC
   VA-01 should show UNAVAILABLE
   VA-02 should handle all operations
   
3. Verify operations continue
   Aggregation/provisioning working
   
4. Restart VA-01
   sudo systemctl start sailpoint-va
   
5. Confirm VA-01 rejoins cluster
   Both VAs show AVAILABLE
```

**Exam Tip:** In HA configuration, if one VA fails, the other(s) take over automatically.

---

## Common Health Issues

### **Issue 1: High CPU Usage**

**Symptoms:**
- CPU consistently >80%
- Operations slow
- Timeouts occurring

**Common Causes:**
- Undersized VA (too small for workload)
- Too many concurrent operations
- Inefficient source queries
- Large aggregations

**Quick Check:**
```bash
# What's using CPU?
top

# Check what's running
tail -f /opt/sailpoint/logs/connector.log
```

### **Issue 2: High Memory Usage**

**Symptoms:**
- Memory >85%
- VA becoming unresponsive
- Out of memory errors

**Common Causes:**
- Memory leak (rare, needs VA update)
- Very large aggregation datasets
- Insufficient RAM for workload

**Quick Check:**
```bash
# Memory usage
free -h

# VA process memory
ps aux | grep va
```

### **Issue 3: Disk Space Low**

**Symptoms:**
- Disk >90% full
- Log errors about disk space
- VA unable to write logs/cache

**Common Causes:**
- Log files not rotating
- Cache not clearing
- Aggregation temporary files

**Quick Fix:**
```bash
# Find large files
du -sh /opt/sailpoint/* | sort -h

# Common culprits:
# /opt/sailpoint/logs - old logs
# /opt/sailpoint/cache - stale cache

# Clean old logs (example - be careful!)
find /opt/sailpoint/logs -name "*.log.*" -mtime +30 -delete

# Clear cache (VA must be stopped)
sudo systemctl stop sailpoint-va
rm -rf /opt/sailpoint/cache/*
sudo systemctl start sailpoint-va
```

**Exam Tip:** Low disk space is often caused by log accumulation. Check `/opt/sailpoint/logs/`.

---

## Practice Exercise

**Question:** A VA shows UNAVAILABLE in ISC. The last heartbeat was 10 minutes ago. What should you check first?

<details>
<summary>Answer</summary>

**Systematic troubleshooting order:**

**1. Check VA Service (Most Common Issue)**
```bash
# Is VA running?
sudo systemctl status sailpoint-va

# If stopped:
sudo systemctl start sailpoint-va

# Check if it stays running:
sudo systemctl status sailpoint-va
```

**2. Check Network Connectivity**
```bash
# Can VA reach ISC?
curl -I https://yourcompany.identitynow.com

# If fails, check:
# - Internet connectivity
# - Firewall rules (outbound 443)
# - DNS resolution
```

**3. Check VA Logs**
```bash
# Why did it disconnect?
tail -100 /opt/sailpoint/logs/ccg.log

# Look for:
# - Connection errors
# - Certificate issues
# - Network timeouts
```

**4. Check System Resources**
```bash
# Out of memory/disk?
free -h
df -h

# If disk full or memory exhausted:
# - Free up space
# - Restart VA
```

**Common Causes:**
- VA service stopped (manual or crash)
- Network/firewall issue
- Certificate expired
- Out of system resources

**Expected Resolution Time:**
- If just service stopped: 1-2 minutes (restart)
- If network issue: Depends on network team
- If certificate issue: Contact SailPoint support
</details>

---

# 2.2.2 Understanding VA Logs

**Learning Objective:** Read and interpret VA log files for operational insight and troubleshooting.

---

## VA Log Files Overview

### **Primary Log Locations**
```
/opt/sailpoint/logs/

Key Files:
├── va.log              (VA service events)
├── ccg.log             (ISC communication)
├── connector.log       (Source operations)
└── charon.log          (Internal connector framework)

Rotated Logs:
├── va.log.1           (Yesterday)
├── va.log.2.gz        (2 days ago, compressed)
└── va.log.30.gz       (30 days ago)
```

**Exam Tip:** Know the three main logs - `va.log`, `ccg.log`, and `connector.log`.

---

## Log Files Explained

### **1. va.log - VA Service Log**

**Purpose:** VA service lifecycle and system events

**What's logged:**
- VA startup/shutdown
- Configuration changes
- System health events
- Service errors
- Resource warnings

**Example entries:**
```
2024-02-13 08:00:15 INFO  [VA] Service starting...
2024-02-13 08:00:16 INFO  [VA] Configuration loaded from /opt/sailpoint/config/va.conf
2024-02-13 08:00:17 INFO  [VA] Connectors initialized: AD, JDBC, LDAP
2024-02-13 08:00:18 INFO  [VA] Service started successfully
2024-02-13 08:00:19 INFO  [VA] Connecting to ISC: yourcompany.identitynow.com
2024-02-13 08:00:20 INFO  [VA] Connection established

2024-02-13 14:30:45 WARN  [VA] High memory usage: 78%
2024-02-13 18:45:12 ERROR [VA] Failed to load connector: CustomConnector
```

**When to check:**
- VA won't start
- Configuration issues
- System resource problems
- Service crashes

### **2. ccg.log - Cloud Gateway Log**

**Purpose:** Communication between VA and ISC

**What's logged:**
- Connection status to ISC
- Heartbeat messages
- Commands received from ISC
- Results sent to ISC
- Authentication events

**Example entries:**
```
2024-02-13 08:00:20 INFO  [CCG] Connected to ISC: yourcompany.identitynow.com
2024-02-13 08:00:21 INFO  [CCG] Authentication successful
2024-02-13 08:01:20 DEBUG [CCG] Heartbeat sent
2024-02-13 08:01:21 DEBUG [CCG] Heartbeat acknowledged

2024-02-13 10:00:00 INFO  [CCG] Command received: ACCOUNT_AGGREGATION
2024-02-13 10:00:01 INFO  [CCG] Command ID: cmd-12345, Source: source-ad-01
2024-02-13 10:05:42 INFO  [CCG] Result sent: cmd-12345, Status: SUCCESS

2024-02-13 15:30:15 ERROR [CCG] Connection lost to ISC
2024-02-13 15:30:45 INFO  [CCG] Reconnecting...
2024-02-13 15:30:50 INFO  [CCG] Connection re-established
```

**When to check:**
- VA shows UNAVAILABLE in ISC
- Connection issues
- Commands not executing
- Heartbeat problems

### **3. connector.log - Connector Operations**

**Purpose:** Source connector activities (aggregation, provisioning)

**What's logged:**
- Aggregation start/completion
- Provisioning operations
- Source connection events
- Record counts
- Operation errors

**Example entries:**
```
2024-02-13 10:00:00 INFO  [Connector] Account aggregation started: Active Directory
2024-02-13 10:00:01 INFO  [AD-Connector] Connecting to DC: dc01.company.com:389
2024-02-13 10:00:02 INFO  [AD-Connector] Connected successfully
2024-02-13 10:00:03 INFO  [AD-Connector] Fetching users from OU=Users,DC=company,DC=com
2024-02-13 10:02:15 INFO  [AD-Connector] Fetched 5,247 user accounts
2024-02-13 10:02:30 INFO  [Connector] Sending accounts to ISC...
2024-02-13 10:05:42 INFO  [Connector] Aggregation completed: 5,247 accounts in 5m 42s

2024-02-13 11:15:23 INFO  [Connector] Provisioning operation: CREATE account
2024-02-13 11:15:24 INFO  [AD-Connector] Creating account: CN=John Smith,OU=Users,DC=company,DC=com
2024-02-13 11:15:26 INFO  [AD-Connector] Account created successfully
2024-02-13 11:15:27 INFO  [Connector] Provisioning completed: SUCCESS

2024-02-13 14:30:45 ERROR [AD-Connector] Connection timeout to dc01.company.com
2024-02-13 14:30:46 ERROR [Connector] Aggregation failed: Active Directory
```

**When to check:**
- Aggregation failures
- Provisioning errors
- Source connection issues
- Performance problems
- Data quality issues

**Exam Tip:** `connector.log` contains ALL aggregation and provisioning operation details.

---

## Log Levels

### **Understanding Log Levels**

| Level | Purpose | Volume | When to Use |
|-------|---------|--------|-------------|
| **ERROR** | Critical failures | Low | Always enabled |
| **WARN** | Potential issues | Low | Always enabled |
| **INFO** | Normal operations | Medium | Default setting |
| **DEBUG** | Detailed trace | High | Troubleshooting only |

**Default:** INFO level (shows INFO, WARN, ERROR)

**Example - Same operation at different levels:**
```
ERROR level (critical only):
[ERROR] Aggregation failed: Connection timeout

WARN level (includes warnings):
[WARN] High memory usage: 78%
[ERROR] Aggregation failed: Connection timeout

INFO level (normal operations):
[INFO] Aggregation started: Active Directory
[WARN] High memory usage: 78%
[ERROR] Aggregation failed: Connection timeout

DEBUG level (everything):
[DEBUG] Connecting to DC: dc01.company.com
[DEBUG] LDAP bind successful
[DEBUG] Query: (&(objectClass=user)(!(objectClass=computer)))
[DEBUG] Fetched 100 records...
[DEBUG] Fetched 200 records...
[INFO] Aggregation started: Active Directory
[WARN] High memory usage: 78%
[ERROR] Aggregation failed: Connection timeout
```

**Exam Tip:** DEBUG logging creates large log files. Use only for troubleshooting, then disable.

---

## Reading Logs Effectively

### **Common Log Reading Commands**

**View current log:**
```bash
# Last 100 lines
tail -100 /opt/sailpoint/logs/connector.log

# Follow log in real-time
tail -f /opt/sailpoint/logs/connector.log

# View with timestamps
tail -f /opt/sailpoint/logs/connector.log | grep "2024-02-13 10:"
```

**Search logs:**
```bash
# Find errors
grep -i "error" /opt/sailpoint/logs/connector.log

# Find specific source
grep "Active Directory" /opt/sailpoint/logs/connector.log

# Find errors in last hour
grep "$(date +%Y-%m-%d\ %H:)" /opt/sailpoint/logs/connector.log | grep ERROR

# Count errors
grep -c "ERROR" /opt/sailpoint/logs/connector.log
```

**Search rotated logs:**
```bash
# Search compressed logs
zgrep "error" /opt/sailpoint/logs/connector.log.*.gz

# Search all logs (current + rotated)
grep "error" /opt/sailpoint/logs/connector.log*
```

**Context around errors:**
```bash
# Show 10 lines before and after error
grep -A 10 -B 10 "ERROR" /opt/sailpoint/logs/connector.log

# Helpful to see what led to error and what happened after
```

---

## Log Rotation

### **How Log Rotation Works**

**Automatic rotation:**
```
Daily at midnight:
  connector.log       → connector.log.1
  connector.log.1     → connector.log.2
  connector.log.2     → connector.log.3.gz (compressed)
  ...
  connector.log.29.gz → connector.log.30.gz
  connector.log.30.gz → Deleted
  
  New connector.log created
```

**Retention:** 30 days (default)

**Compression:** After 2-3 days, gzipped to save space

**Configuration:** `/opt/sailpoint/config/logrotate.conf` (usually)

---

## Common Log Patterns

### **Successful Aggregation**
```
[INFO] Account aggregation started: Active Directory
[INFO] Connecting to dc01.company.com:389
[INFO] Connected successfully
[INFO] Fetching accounts...
[INFO] Fetched 10,247 accounts
[INFO] Sending to ISC...
[INFO] Aggregation completed: 10,247 accounts in 8m 15s
```

**Key indicators:**
- "started" → "connected" → "fetched" → "completed"
- Record count matches expectations
- Reasonable time duration
- No errors

### **Failed Aggregation**
```
[INFO] Account aggregation started: Active Directory
[INFO] Connecting to dc01.company.com:389
[ERROR] Connection timeout after 30 seconds
[ERROR] Failed to connect to source
[ERROR] Aggregation failed: Active Directory
[INFO] Reporting failure to ISC
```

**Key indicators:**
- ERROR between start and completion
- Specific error message (connection timeout)
- "failed" status
- Failure reported to ISC

### **Successful Provisioning**
```
[INFO] Provisioning operation received: CREATE account
[INFO] Target: Active Directory, Account: jsmith
[INFO] Connecting to dc01.company.com
[INFO] Creating CN=jsmith,OU=Users,DC=company,DC=com
[INFO] Account created successfully
[INFO] Setting attributes: mail, displayName, department
[INFO] Adding to groups: CN=All-Users,OU=Groups,DC=company,DC=com
[INFO] Provisioning completed: SUCCESS
[INFO] Sending result to ISC
```

### **Failed Provisioning**
```
[INFO] Provisioning operation received: CREATE account
[INFO] Target: Active Directory, Account: jsmith
[INFO] Connecting to dc01.company.com
[INFO] Creating CN=jsmith,OU=Users,DC=company,DC=com
[ERROR] LDAP Error: ENTRY_ALREADY_EXISTS
[ERROR] Account jsmith already exists in Active Directory
[ERROR] Provisioning failed: ENTRY_ALREADY_EXISTS
[INFO] Sending failure result to ISC
```

**Exam Tip:** ERROR messages usually include error code (ENTRY_ALREADY_EXISTS, CONNECTION_TIMEOUT, etc.).

---

## Practice Exercise

**Question:** You need to find out why an Active Directory aggregation failed yesterday at 2 AM. Which log file and commands would you use?

<details>
<summary>Answer</summary>

**Log file:** `/opt/sailpoint/logs/connector.log` (or rotated version)

**Steps:**

**1. Determine which log file has yesterday's data**
```bash
# If today is Feb 14, yesterday (Feb 13) might be in:
# - connector.log (if it's early today)
# - connector.log.1 (if rotation already happened)

# Check file timestamps
ls -lh /opt/sailpoint/logs/connector.log*
```

**2. Search for the failed aggregation**
```bash
# Search for Active Directory aggregation around 2 AM on Feb 13
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log.1 | grep -i "active directory"

# Or search for aggregation failure
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log.1 | grep -i "aggregation failed"
```

**3. Get context around the error**
```bash
# Show 20 lines before and after the failure
grep -A 20 -B 20 "Aggregation failed.*Active Directory" /opt/sailpoint/logs/connector.log.1
```

**4. Look for ERROR messages in that timeframe**
```bash
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log.1 | grep ERROR
```

**Example output might show:**
```
2024-02-13 02:00:00 INFO  Aggregation started: Active Directory
2024-02-13 02:00:01 INFO  Connecting to dc01.company.com:389
2024-02-13 02:00:31 ERROR Connection timeout after 30 seconds
2024-02-13 02:00:32 ERROR Failed to connect to source
2024-02-13 02:00:32 ERROR Aggregation failed: Active Directory
```

**Root cause:** Connection timeout to domain controller

**Next steps:**
- Check network connectivity to DC
- Verify DC was running at 2 AM
- Check if firewall blocked connection
- Try manual aggregation to see if issue persists
</details>

---

# 2.2.3 Basic VA Troubleshooting Techniques

**Learning Objective:** Diagnose and resolve common Virtual Appliance issues.

---

## Systematic Troubleshooting Approach

### **The 5-Step Method**
```
1. IDENTIFY the symptom
   What's the actual problem?
   
2. CHECK the obvious
   Service running? Network connected? Disk full?
   
3. READ the logs
   What do the logs say happened?
   
4. ISOLATE the cause
   Is it VA, network, or source?
   
5. FIX and verify
   Apply fix, test, monitor
```

**Exam Tip:** Always check basics first - service status, logs, connectivity.

---

## Common Issue #1: VA Shows UNAVAILABLE

### **Symptom**
```
ISC UI shows:
  VA-PROD-01: ● UNAVAILABLE (Red)
  Last Heartbeat: 15 minutes ago
```

### **Troubleshooting Steps**

**Step 1: Check VA Service**
```bash
# Is VA running?
sudo systemctl status sailpoint-va

# If "inactive (dead)":
sudo systemctl start sailpoint-va

# If it won't start or keeps stopping:
# Check va.log for why
tail -50 /opt/sailpoint/logs/va.log
```

**Step 2: Check Network Connectivity**
```bash
# Can VA reach ISC?
curl -I https://yourcompany.identitynow.com

# Test DNS resolution
nslookup yourcompany.identitynow.com

# Test port 443
telnet yourcompany.identitynow.com 443
# or
nc -zv yourcompany.identitynow.com 443
```

**Step 3: Check CCG Logs**
```bash
# Why can't VA connect to ISC?
tail -100 /opt/sailpoint/logs/ccg.log | grep -i error

# Common errors:
# - "Connection refused" = ISC unreachable
# - "Certificate error" = Cert expired/invalid
# - "Authentication failed" = Keyfile issue
```

**Step 4: Check Certificate**
```bash
# Certificate location
ls -l /opt/sailpoint/config/certificates/

# Check expiration (if accessible)
openssl x509 -in /opt/sailpoint/config/certificates/va-cert.pem -noout -dates
```

**Common Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| VA service stopped | `sudo systemctl start sailpoint-va` |
| Network/firewall blocking | Work with network team to allow outbound 443 |
| Certificate expired | Contact SailPoint support for renewal |
| ISC outage | Check SailPoint status page, wait for resolution |
| Out of resources (memory/disk) | Free resources, restart VA |

---

## Common Issue #2: Aggregation Fails

### **Symptom**
```
ISC shows:
  Active Directory aggregation: FAILED
  Last successful: 2 days ago
```

### **Troubleshooting Steps**

**Step 1: Check Connector Logs**
```bash
# Find the failed aggregation
grep -i "aggregation failed" /opt/sailpoint/logs/connector.log | tail -5

# Get details around the failure
grep -A 20 "Active Directory" /opt/sailpoint/logs/connector.log | grep -A 20 "aggregation started" | tail -30
```

**Step 2: Identify the Error**

**Common aggregation errors:**

**Connection Timeout:**
```
ERROR: Connection timeout to dc01.company.com
```
**Solution:**
- Verify domain controller is online
- Check network connectivity: `ping dc01.company.com`
- Verify firewall rules allow LDAP (389/636)
- Check if DC is overloaded

**Authentication Failed:**
```
ERROR: LDAP bind failed: Invalid credentials
```
**Solution:**
- Verify service account password hasn't changed
- Check account is not locked/disabled
- Test credentials: `ldapsearch -x -H ldap://dc01.company.com -D "svc_sailpoint@company.com" -W`
- Update credentials in ISC if changed

**Insufficient Permissions:**
```
ERROR: Access denied reading OU=Users,DC=company,DC=com
```
**Solution:**
- Verify service account has read permissions
- Check OU permissions in AD
- Grant necessary permissions to service account

**Source Unreachable:**
```
ERROR: Name or service not known: dc01.company.com
```
**Solution:**
- Check DNS: `nslookup dc01.company.com`
- Verify hostname/IP correct
- Check network connectivity

**Step 3: Test Source Connection**

**Manual LDAP test (AD):**
```bash
# Test basic connectivity
ldapsearch -x -H ldap://dc01.company.com -b "DC=company,DC=com" -D "svc_sailpoint@company.com" -W "(objectClass=user)" sAMAccountName | head -20

# If successful: Returns user accounts
# If fails: Shows error (auth, connection, permissions)
```

**Manual database test (JDBC):**
```bash
# Install SQL client if not present
# Test connection
sqlplus svc_sailpoint/password@hostname:1521/dbname

# Or for MySQL:
mysql -h hostname -u svc_sailpoint -p database_name
```

**Step 4: Review ISC Source Configuration**

1. Log into ISC
2. Go to **Admin** → **Connections** → **Sources**
3. Find your source (e.g., Active Directory)
4. Click **Edit**
5. Verify:
   - Hostname/IP correct
   - Port correct (389 for LDAP, 636 for LDAPS)
   - Service account username correct
   - Test connection (button in UI)

**Exam Tip:** Always check source credentials haven't changed and service account isn't locked.

---

## Common Issue #3: Provisioning Fails

### **Symptom**
```
ISC shows:
  Create AD account for jsmith: FAILED
  Error: Account creation failed
```

### **Troubleshooting Steps**

**Step 1: Find Provisioning Error in Logs**
```bash
# Search for the failed provisioning
grep -i "jsmith" /opt/sailpoint/logs/connector.log | grep -i "error"

# Or search by time if you know when it failed
grep "2024-02-13 14:" /opt/sailpoint/logs/connector.log | grep -i "provision"
```

**Step 2: Identify Error Type**

**Account Already Exists:**
```
ERROR: LDAP Error: ENTRY_ALREADY_EXISTS
Account CN=jsmith,OU=Users,DC=company,DC=com already exists
```
**Solution:**
- Check if account truly exists in AD
- If exists: Correlate account to identity in ISC
- If ghost entry: Delete from AD, retry

**Insufficient Permissions:**
```
ERROR: LDAP Error: INSUFFICIENT_ACCESS_RIGHTS
Unable to create account in OU=Users,DC=company,DC=com
```
**Solution:**
- Verify service account has create permissions
- Check delegation on target OU
- Grant necessary permissions in AD

**Invalid Attribute:**
```
ERROR: LDAP Error: INVALID_ATTRIBUTE_VALUE
Attribute 'telephoneNumber' value 'N/A' violates schema
```
**Solution:**
- Check attribute value format
- Fix in ISC provisioning policy
- Update attribute mapping/transform

**OU Doesn't Exist:**
```
ERROR: LDAP Error: NO_SUCH_OBJECT
OU=NewHires,OU=Users,DC=company,DC=com not found
```
**Solution:**
- Create OU in AD
- Or fix OU path in provisioning policy

**Step 3: Test Manually in AD**
```powershell
# Try creating account manually with same attributes
New-ADUser -Name "John Smith" -SamAccountName "jsmith" -Path "OU=Users,DC=company,DC=com"

# If this fails with same error:
# = AD issue, not ISC issue

# If this succeeds:
# = ISC provisioning policy issue
```

**Step 4: Check Provisioning Policy in ISC**

1. Go to **Admin** → **Connections** → **Sources**
2. Click source → **Provisioning** tab
3. Review provisioning policy
4. Check **Create** operation configuration
5. Verify attribute mappings correct

**Exam Tip:** ENTRY_ALREADY_EXISTS usually means account exists or correlation needed.

---

## Common Issue #4: High CPU/Memory Usage

### **Symptom**
```
VA showing:
  CPU: 95%
  Memory: 88%
  Operations slow or timing out
```

### **Troubleshooting Steps**

**Step 1: Check What's Running**
```bash
# What's consuming resources?
top

# Look for:
# - java process (VA/connectors)
# - High CPU% or MEM%

# Detailed view
ps aux | grep -i sailpoint | grep -v grep
```

**Step 2: Check Current Operations**
```bash
# What operations are active?
tail -100 /opt/sailpoint/logs/connector.log

# Is large aggregation running?
# Multiple aggregations concurrent?
# Stuck operation?
```

**Step 3: Identify Cause**

**Undersized VA:**
- 50K users but only 4 CPU / 8 GB RAM
- Solution: Increase VA resources

**Too Many Concurrent Operations:**
- Multiple aggregations scheduled at same time
- Solution: Stagger schedules, run off-peak

**Memory Leak (Rare):**
- Memory keeps climbing, never releases
- Solution: Restart VA, update to latest version

**Large Dataset:**
- Aggregating 100K+ users with many attributes
- Solution: Use delta aggregation, increase resources

**Step 4: Immediate Mitigation**
```bash
# If VA is unresponsive:
# 1. Check if operation can be stopped in ISC

# 2. If VA completely hung:
sudo systemctl restart sailpoint-va

# 3. Monitor restart:
tail -f /opt/sailpoint/logs/va.log
```

**Step 5: Long-term Fix**

- Resize VA if undersized
- Optimize aggregation schedules
- Use delta aggregation instead of full
- Add second VA for load distribution
- Filter unnecessary attributes in aggregation

---

## Common Issue #5: Disk Space Full

### **Symptom**
```
Disk: 98% full
VA logs show: "No space left on device"
```

### **Troubleshooting Steps**

**Step 1: Check Disk Usage**
```bash
# Overall disk usage
df -h

# What's using space in /opt/sailpoint?
du -sh /opt/sailpoint/*

# Common culprits:
# /opt/sailpoint/logs - 15 GB (logs not rotating)
# /opt/sailpoint/cache - 8 GB (stale cache)
```

**Step 2: Identify Large Files**
```bash
# Find largest files
find /opt/sailpoint -type f -size +100M -exec ls -lh {} \;

# Find old logs
find /opt/sailpoint/logs -name "*.log" -mtime +30

# Find large logs
ls -lhS /opt/sailpoint/logs/ | head -20
```

**Step 3: Clean Up (CAREFULLY)**
```bash
# DO NOT delete current logs (*.log)
# ONLY delete rotated/archived logs if needed

# Delete logs older than 30 days (example)
find /opt/sailpoint/logs -name "*.log.*" -mtime +30 -delete

# Delete old compressed logs
find /opt/sailpoint/logs -name "*.gz" -mtime +60 -delete

# Clear cache (VA must be stopped first!)
sudo systemctl stop sailpoint-va
rm -rf /opt/sailpoint/cache/*
sudo systemctl start sailpoint-va
```

**Step 4: Prevent Recurrence**

- Verify log rotation is working
- Monitor disk space proactively
- Set up alerts for >80% disk usage
- Consider larger disk or separate partition for logs

**Exam Tip:** Always stop VA service before clearing cache directory.

---

## Troubleshooting Toolkit

### **Essential Commands**
```bash
# Service management
sudo systemctl status sailpoint-va
sudo systemctl start sailpoint-va
sudo systemctl stop sailpoint-va
sudo systemctl restart sailpoint-va

# Log viewing
tail -f /opt/sailpoint/logs/va.log
tail -f /opt/sailpoint/logs/ccg.log
tail -f /opt/sailpoint/logs/connector.log
grep -i "error" /opt/sailpoint/logs/connector.log

# System health
top
free -h
df -h
netstat -tulpn | grep va

# Network testing
ping dc01.company.com
curl -I https://yourcompany.identitynow.com
nslookup dc01.company.com
telnet dc01.company.com 389

# Source testing (LDAP/AD)
ldapsearch -x -H ldap://dc01.company.com -D "user@domain" -W -b "dc=company,dc=com"
```

---

## When to Contact SailPoint Support

**Contact support for:**
- VA won't start after troubleshooting
- Certificate expired/invalid
- Suspected VA software bug
- Need VA version upgrade
- Connector-specific issues beyond basic troubleshooting
- ISC outage or service degradation

**Before contacting support, gather:**
```
☐ VA logs (va.log, ccg.log, connector.log)
☐ Error messages (exact text)
☐ VA version (from ISC UI or logs)
☐ Tenant name
☐ Source configuration (screenshots)
☐ Steps to reproduce issue
☐ When issue started
☐ What changed recently
```

**Exam Tip:** Gather logs and details before contacting support. They'll need this information.

---

## Practice Exercises

**Exercise 1:** An aggregation fails with "Connection timeout to dc01.company.com". What would you check?

<details>
<summary>Answer</summary>

**Systematic checks:**

**1. Network Connectivity**
```bash
# Can VA reach DC?
ping dc01.company.com

# Is DNS working?
nslookup dc01.company.com

# Is LDAP port open?
telnet dc01.company.com 389
# or
nc -zv dc01.company.com 389
```

**2. Domain Controller Status**
- Is DC online and responding?
- Is DC overloaded (high CPU/memory)?
- Is LDAP service running on DC?

**3. Firewall Rules**
- Is firewall blocking VA → DC connection?
- Is port 389 (LDAP) or 636 (LDAPS) open?
- Check both VA and DC firewalls

**4. VA Network Configuration**
- Is VA's network interface up?
- Is routing table correct?
- Can VA reach other systems?

**5. Timeout Settings**
- Is timeout too short for large queries?
- Check connector timeout configuration

**Common Causes:**
- DC offline or unreachable
- Firewall blocking LDAP port
- Network issue between VA and DC
- DC overloaded and responding slowly

**Quick Test:**
```bash
# If this works, connection is fine:
ldapsearch -x -H ldap://dc01.company.com -b "dc=company,dc=com" "(objectClass=user)" -D "svc@domain" -W

# If this fails with timeout:
# = Network/DC issue, not ISC configuration
```
</details>

---

**Exercise 2:** VA logs show "ERROR: ENTRY_ALREADY_EXISTS" when creating account jsmith. What's the issue and solution?

<details>
<summary>Answer</summary>

**Issue:** Account jsmith already exists in Active Directory

**Possible Scenarios:**

**Scenario 1: Legitimate existing account**
- User had account before, wasn't removed
- Account exists but not correlated in ISC

**Solution:**
1. Verify account exists in AD:
```powershell
   Get-ADUser jsmith
```

2. In ISC, correlate existing AD account to identity:
   - Go to identity's accounts
   - Use "Correlate Account" function
   - Match to existing jsmith account

**Scenario 2: Ghost/orphaned account**
- Account partially created in previous failed attempt
- Account exists in AD but shouldn't

**Solution:**
1. Delete account in AD:
```powershell
   Remove-ADUser jsmith
```

2. Retry provisioning in ISC

**Scenario 3: Duplicate username**
- Two different people trying to use same username
- Naming convention collision

**Solution:**
- Use different username (jsmith2, john.smith, etc.)
- Update ISC naming transform to avoid collisions

**Prevention:**
- Enable account correlation in ISC
- Use unique username generation
- Check for existing accounts before creating

**Exam Tip:** ENTRY_ALREADY_EXISTS = account exists, usually needs correlation not re-creation.
</details>

---

**Exercise 3:** You need to troubleshoot a provisioning failure that happened 3 hours ago. Where do you look and what commands do you use?

<details>
<summary>Answer</summary>

**Step-by-Step Investigation:**

**1. Determine Exact Time**
```
If current time is 14:00 and issue was 3 hours ago:
  Look for logs around 11:00
```

**2. Check Connector Log**
```bash
# Search provisioning around 11:00
grep "2024-02-13 11:" /opt/sailpoint/logs/connector.log | grep -i "provision"

# Look for ERROR messages in that timeframe
grep "2024-02-13 11:" /opt/sailpoint/logs/connector.log | grep ERROR

# Get context around the error
grep -A 15 -B 5 "Provisioning.*failed" /opt/sailpoint/logs/connector.log | grep -A 20 "2024-02-13 11:"
```

**3. Identify What Failed**
```
Look for:
- Which account? (username/DN)
- Which operation? (CREATE, MODIFY, DELETE)
- Which source? (Active Directory, database, etc.)
- What error? (error code and message)
```

**4. Example Output**
```
2024-02-13 11:15:00 INFO  Provisioning operation: CREATE account
2024-02-13 11:15:01 INFO  Target: Active Directory, Account: jsmith
2024-02-13 11:15:02 ERROR LDAP Error: ENTRY_ALREADY_EXISTS
2024-02-13 11:15:02 ERROR Account jsmith already exists
2024-02-13 11:15:03 INFO  Provisioning failed: ENTRY_ALREADY_EXISTS
```

**5. Also Check ISC UI**
```
In ISC:
  Admin → Activity → Provisioning
  Filter: Time = 11:00-12:00, Status = Failed
  Click on failed operation
  View details and error message
```

**6. Correlate Logs with ISC**
```
Connector log shows: Error at 11:15:02
ISC UI shows: Same error at 11:15:03
= Confirms this is the issue
```

**Key Files:**
- `/opt/sailpoint/logs/connector.log` (primary)
- `/opt/sailpoint/logs/va.log` (if service issue)
- ISC UI Activity logs (for additional context)
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Monitoring VA Health:**
- Healthy VA: CPU <60%, Memory <70%, Disk >20% free, Connected to ISC
- Heartbeat: Every 60 seconds, 3+ missed = UNAVAILABLE
- Check status: ISC UI (Admin → Virtual Appliances)
- Check system: `systemctl status sailpoint-va`, `top`, `df -h`

✅ **VA Logs:**
- **va.log**: Service lifecycle, system events
- **ccg.log**: ISC communication, heartbeat, connection
- **connector.log**: Aggregation, provisioning, source operations
- Location: `/opt/sailpoint/logs/`
- Log levels: ERROR, WARN, INFO, DEBUG

✅ **Troubleshooting Approach:**
1. Check service status first
2. Review relevant logs
3. Test connectivity (network, source)
4. Identify error type
5. Apply solution, verify fix

✅ **Common Issues:**
- VA UNAVAILABLE: Check service, network, logs
- Aggregation fails: Check connector.log, test source connection
- Provisioning fails: Common errors (ENTRY_ALREADY_EXISTS, INSUFFICIENT_ACCESS)
- High CPU/Memory: Check running operations, resize if needed
- Disk full: Clean old logs, check rotation

✅ **Key Commands:**
```bash
# Service
sudo systemctl status sailpoint-va

# Logs
tail -f /opt/sailpoint/logs/connector.log
grep -i "error" /opt/sailpoint/logs/connector.log

# System
top, free -h, df -h

# Network
ping <host>, curl -I <url>
```

**Exam Focus Areas:**
- Know three main log files (va.log, ccg.log, connector.log)
- Understand heartbeat mechanism (60s intervals)
- Common error codes (ENTRY_ALREADY_EXISTS, CONNECTION_TIMEOUT)
- Systematic troubleshooting approach
- When to contact SailPoint support

---

**End of Section 2: Virtual Appliances**

**Next:** Section 3: Sources (Data Integration)

---

**Study Tips:**
- Practice reading actual logs to recognize patterns
- Know log file locations by heart
- Understand common errors and their solutions
- Master basic Linux commands for troubleshooting
- Remember: Check service status and logs FIRST
