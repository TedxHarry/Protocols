# Virtual Appliance — What It Is, How It Works, and Why It Matters

ISC is a cloud platform. Your Active Directory domain controllers are not on the cloud — they're in a datacenter in New Jersey or a server room in the building basement. Your legacy ERP runs on a server that predates the term "cloud native" by a decade. Your Oracle database does not have a public API endpoint.

The Virtual Appliance (VA) is what makes ISC work anyway.

Understanding the VA is not optional for an integration engineer. Every on-premises integration you design runs through it. Every connectivity problem with on-premises systems eventually becomes a VA investigation. VA sizing, VA high availability, and VA health monitoring are part of every enterprise ISC deployment.

---

## What the Virtual Appliance Actually Is

The VA is a Linux-based virtual machine that runs inside your network, registers with your ISC tenant, and acts as a secure outbound proxy between ISC cloud and your on-premises systems.

The key word is outbound. The VA initiates all connections — it calls out to ISC cloud over HTTPS on port 443. ISC cloud never initiates a connection inward to your network. This design means:

- No inbound firewall rules needed
- No public IP needed for the VA itself
- No DMZ configuration
- The VA sits inside your network perimeter, with regular outbound HTTPS access (same as a browser)

```
Your Network
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Active Directory ◄──────────────────┐                         │
│                                      │                         │
│  Oracle DB ◄─────────────────────────┤                         │
│                                      ▼                         │
│  Legacy ERP ◄──────────────── Virtual Appliance ──HTTPS:443──► │ ISC Cloud
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

The VA registers with your ISC tenant using a cluster key generated in the ISC admin console. After registration, it maintains a persistent connection with ISC cloud. When ISC needs to aggregate data from Active Directory, it sends the aggregation instruction through that persistent connection to the VA, which runs the actual aggregation against your domain controller and returns the results.

---

## VA Clusters

A single VA is not a production architecture. If that VM fails or needs maintenance, every on-premises connector it serves stops working.

ISC addresses this with VA clusters. A cluster is a group of two or more VAs that share the same cluster key, same ISC configuration, and same connected systems. ISC distributes work across the VAs in a cluster — if one VA is unavailable, ISC routes operations to the others automatically.

**Minimum production architecture**: Two VAs per cluster. One can handle maintenance windows and unexpected failures while the other keeps aggregations running.

**Scaling**: Additional VAs added to a cluster for throughput. If you have 50 on-premises systems and need simultaneous aggregation windows, a two-VA cluster may bottleneck. Add VAs until throughput meets your aggregation schedule.

**Cluster management**: All VAs in a cluster should run the same software version. ISC supports rolling upgrades — upgrade one VA, verify it's healthy, upgrade the next. Never upgrade all VAs simultaneously; you'd lose connectivity during the upgrade window.

> ⚠️ **COMMON MISTAKE:** One VA in production. The most common VA architecture mistake — and the one that causes the most midnight pages. When the single VA goes down (hardware failure, corrupted VM, full disk), every on-premises aggregation stops until someone recovers it. Two VAs minimum. This is not a recommendation; it's the configuration that SailPoint themselves will tell you is required for production.

---

## VA Deployment Options

**OVA (VMware/vSphere)**: The most common deployment method. SailPoint provides an OVA file you import into your VMware environment. After import, boot the VM, configure basic networking (IP, DNS, proxy if needed), run the setup script that connects it to your ISC tenant, and you're done.

**VHD (Hyper-V)**: Same concept as OVA but for Windows Hyper-V environments. Download the VHD image, create a new VM in Hyper-V Manager using the provided disk, complete setup.

**Docker/container**: Newer deployment option. The VA ships as a container image. Deploy in your container infrastructure (Docker, Kubernetes). Container deployment is more operationally flexible — fits into existing CI/CD pipelines for VM management — but requires container infrastructure knowledge.

**Sizing requirements** (current as of 2025):

| Deployment Size | CPUs | RAM | Disk |
|---|---|---|---|
| Standard (< 50,000 identities) | 4 vCPU | 16 GB | 100 GB |
| Large (50,000–200,000 identities) | 8 vCPU | 32 GB | 200 GB |
| Enterprise (200,000+ identities) | 16 vCPU | 64 GB | 400 GB |

> 💡 **REMEMBER:** These are minimums, not targets. Memory pressure on a VA causes slow aggregations, timeouts, and eventually connector failures. If your VA is regularly running aggregations for 20+ systems simultaneously, size up. The cost of over-sizing a VM is a few hundred dollars per year. The cost of a VA that crashes during the nightly aggregation window is an incident.

---

## Network Requirements

The VA needs outbound HTTPS access to ISC cloud endpoints. The specific domains vary by ISC tenant region (US, EU, etc.) but always follow `*.sailpoint.com` and `*.identitynow.com` patterns. ISC provides a current list in the deployment documentation.

**Proxy support**: If your network routes outbound traffic through an HTTP/HTTPS proxy, the VA supports proxy configuration. Configure the proxy host, port, and credentials during VA setup. One common failure: proxy credentials expire or rotate, and nobody updates the VA configuration — VA loses connectivity, aggregations fail, operations team gets alerted about AD not syncing.

**DNS**: The VA needs to resolve both internal systems (your domain controllers, application servers) and external SailPoint cloud endpoints. Verify DNS works from the VA host before troubleshooting anything else — most early VA connectivity problems are DNS.

**Firewall requirements** (outbound only):

| Destination | Port | Protocol | Purpose |
|---|---|---|---|
| `*.sailpoint.com` | 443 | HTTPS | ISC cloud connection |
| `*.identitynow.com` | 443 | HTTPS | ISC cloud connection |
| Internal AD domain controllers | 389/636 | LDAP/LDAPS | AD aggregation |
| Internal application servers | App-specific | Varies | Other connectors |

---

## VA Health States

The ISC admin console displays each VA's health state. Understanding what each state means tells you where to look when things break:

**Active**: VA is connected to ISC cloud, receiving jobs, processing normally. Green.

**Connected**: VA is connected but not currently processing jobs. Normal when no aggregation is running.

**Degraded**: VA is connected but reporting problems — typically high memory, CPU saturation, or one of its connected systems is unreachable. Jobs may be slow or timing out. Investigate immediately; degraded VAs often precede failures.

**Disconnected**: VA is not connected to ISC cloud. Either the VM is down, the network path is broken, or the VA service has crashed. All on-premises aggregations from this VA stop until connectivity is restored.

> 🎯 **KEY POINT:** Set up VA health monitoring alerts before your first production aggregation. ISC can send email or webhook alerts when VA health changes. Without monitoring, a disconnected VA may go unnoticed for hours — especially on weekends when nobody is watching the admin console.

---

## VA Logs and Troubleshooting

When a VA-connected integration fails, the investigation starts with the VA logs. The VA writes logs to `/home/sailpoint/log/` by default.

**Key log files**:

| Log File | Contains |
|---|---|
| `ccg.log` | Connector and aggregation operations — the primary troubleshooting log |
| `va.log` | VA service startup, registration, cloud connectivity |
| `charon.log` | Authentication and credential management |

**Common VA problems and where to look**:

*Aggregation failing with "connection refused"*: The VA can't reach the target system. Check network path from VA host to the system. Firewall rule change? System IP changed? LDAP port blocked?

*VA showing Disconnected*: Check `va.log` for the last successful connection timestamp and any error messages. Common causes: network change between VA host and internet, proxy credential rotation, SailPoint certificate update.

*Aggregation timing out*: Check `ccg.log` for timeout messages. Usually means the target system is slow or the VA is under load. Check VA CPU/memory. If VA is healthy, check target system performance.

*VA health showing Degraded*: Check VA resource usage (CPU, memory, disk). If disk is full, the VA stops processing. Clean old log files or expand the disk.

> 🎯 **KEY POINT:** ISC also provides an in-console log viewer for VA operations. Before SSH-ing into the VA itself, check the admin console — the connector error messages are often more readable there than in raw log files.

---

## VA Upgrades

SailPoint releases VA software updates periodically. Updates include security patches, performance improvements, and new connector capabilities.

**Upgrade process** (for a two-VA cluster):
1. Confirm both VAs are Active and healthy
2. Put VA-1 into maintenance mode in ISC console (ISC stops routing jobs to it)
3. Upgrade VA-1 (ISC provides the upgrade script — run it via SSH)
4. Verify VA-1 health returns to Active
5. Repeat for VA-2

**Auto-update option**: ISC supports automatic VA updates during a configured maintenance window. Convenient for organizations that don't have dedicated staff monitoring VA versions, but be aware that auto-update on a single-VA deployment = unmonitored outages.

---

## Key Takeaways

- The VA is a Linux VM that runs inside your network and creates an outbound-only HTTPS connection to ISC cloud — no inbound firewall rules needed
- VA clusters provide high availability — minimum two VAs per cluster for any production deployment
- Size VAs based on identity count and aggregation concurrency — undersized VAs cause timeouts and degraded health
- Monitor VA health states (Active/Connected/Degraded/Disconnected) and set up alerts before go-live
- The primary troubleshooting log is `ccg.log` in `/home/sailpoint/log/` — most connector failures leave evidence there
- Upgrade VAs one at a time (never all simultaneously) to maintain connectivity during maintenance

> 🔗 **CONNECTS TO:** The VA gets your on-premises systems reachable from ISC. The next question is which connector type to use for each system — and that depends on what kind of system it is, what interfaces it exposes, and what ISC needs to do with it.
