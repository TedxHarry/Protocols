# Making Slow Integrations Fast — Performance Optimization

The aggregation that takes 6 hours is not a user problem — it's a design problem. Full aggregations for 500,000-identity environments can run in under 90 minutes when configured correctly. Provisioning operations that take 20 minutes in one environment take under 2 minutes in another with the same connector, the same target system, and different configuration choices.

Performance problems in ISC almost always have identifiable root causes and fixable solutions. This document gives you the diagnostic approach and the optimization techniques that actually work.

---

## Where Time Goes in an ISC Aggregation

Before optimizing, understand where time is actually spent. An aggregation has four phases, each with its own performance profile:

**Phase 1 — Retrieval** (connector reads from source): How fast is the source system's API? How many round-trips does ISC need? Is there pagination?

**Phase 2 — Processing** (transform, validate): How complex are the transforms? How many BeanShell rules run per account? How many attributes are being processed?

**Phase 3 — Correlation** (match accounts to identities): How complex is the correlation logic? Are there expensive operations in the BeanShell rules?

**Phase 4 — Persistence** (write results to ISC database): ISC writes updated account records, identity attributes, and policy evaluation results. Scales with volume.

**Identifying the bottleneck**: Compare aggregation duration to the four phases by watching the ISC aggregation progress indicators. If an aggregation finishes "reading" quickly but spends 80% of its time in "processing," the bottleneck is in transforms or correlation. If reading takes the majority of time, the bottleneck is at the source.

---

## Optimization Category 1: Aggregation Architecture

### Full to Delta Migration

The single highest-impact optimization for large sources: replace frequent full aggregations with delta aggregations.

**Before**: Full aggregation every 4 hours. Every cycle reads all 200,000 accounts, processes all transforms, runs all correlation rules.

**After**: Delta aggregation every 4 hours (reads only the ~500 accounts that changed). Full aggregation weekly (reads all 200,000 accounts, but only once per week).

**Impact**: A 200,000-account full aggregation that takes 4 hours runs in under 5 minutes as a delta (500 accounts × proportional time). Weekly full aggregations validate that nothing was missed; daily operations run on delta.

**Prerequisites for delta**: The source system must support delta/incremental queries — either a modified-timestamp field or a change-tracking mechanism. Most modern systems (Workday, AD, Salesforce) support delta queries. Legacy systems may not.

### Aggregation Schedule Optimization

Running 20 aggregations simultaneously strains both the VA and the connected systems. Stagger them.

**Before**: All 28 sources configured to aggregate at 3:00 AM. VA processes all 28 simultaneously; each takes longer because the VA is CPU- and memory-saturated. Total completion time: 5 hours.

**After**: Stagger start times. High-priority sources (Workday, AD) start at 2:00 AM. Medium-priority at 3:00 AM. Low-priority at 4:00 AM. Each source gets dedicated VA resources during its window. Total completion time: 3 hours for the same sources.

**The cascade timing problem**: If Source A feeds attributes that Source B's correlation rules need, Source A's aggregation must complete before Source B's correlation runs correctly. Build your schedule to respect these dependencies.

---

## Optimization Category 2: VA Configuration

### VA Sizing

An undersized VA is a throttle on every aggregation and provisioning operation it handles. The minimum sizing guidelines from Phase 4.1 are minimums — review actual resource utilization before concluding the VA is correctly sized.

**Diagnosing VA resource pressure**:
- Log into the VA host: `top` shows real-time CPU and memory usage
- During a large aggregation, VA CPU should peak but not pin at 100% for extended periods
- Memory: if VA memory usage exceeds 80% of available RAM during aggregation, swap activity begins and performance degrades significantly

**Scaling options**:
- Increase VA VM resources (CPU, RAM) if the VA is under-provisioned for its load
- Add a second VA to the cluster (distributes load across VAs)
- Consider dedicated VA clusters for high-load sources (AD aggregation VA cluster separate from general-purpose cluster)

### Connection Pool Tuning

The VA maintains connection pools to connected systems. Default connection pool sizes are appropriate for most deployments. For high-concurrency environments (many simultaneous provisioning operations), increasing the connection pool maximum reduces wait time for new operations.

**Configuration**: VA configuration files in `/home/sailpoint/config/` contain connection pool settings. Modify after consulting SailPoint support — these are low-level settings where incorrect values can cause instability.

---

## Optimization Category 3: Source System Performance

ISC's aggregation speed is bounded by how fast the source system responds to queries. A Workday API that takes 3 seconds per page of results produces a slower aggregation than one that takes 0.5 seconds — regardless of ISC configuration.

**Diagnosing source system bottlenecks**: Enable connector debug logging during a slow aggregation. In the `ccg.log`, look for timestamps around API calls to the source system. If the gap between "sending request" and "received response" entries is large (>2 seconds for a simple API call), the bottleneck is on the source system side.

**Options when the source is slow**:

**Reduce page size**: Some APIs respond faster to smaller page requests. If Workday's API times out on 1,000-record pages, try 500-record pages. More total requests, but each succeeds. Configure page size in the source connector settings.

**Off-peak aggregation timing**: Source systems often have lower latency during off-peak hours. Moving a Salesforce aggregation from 12 PM (peak Salesforce usage) to 2 AM (minimal usage) may improve performance by 40%.

**API version upgrade**: Some connector/API version combinations are significantly faster than others. Workday REST API v34+ is faster than the SOAP-based Workday connector that older ISC deployments used. If your connector uses an older API version, evaluate the upgrade path.

**Source system resource allocation**: For on-premises systems, the domain controller handling LDAP queries for ISC's AD aggregation may be under-resourced. Work with the AD team to ensure ISC's preferred DC is appropriately sized and not handling authentication load simultaneously with aggregation queries.

---

## Optimization Category 4: ISC Configuration

### Transform Performance

Complex BeanShell rules run once per account per aggregation. In a 500,000-account aggregation, a rule that takes 50ms runs for 6.9 hours of cumulative rule execution time. Rules that take 5ms run for 41 minutes.

**Profile rule performance before deploying at scale**: Test a rule against 10,000 accounts in a sandbox and measure total processing time. Extrapolate to your full volume.

**Common BeanShell performance issues**:

*External calls inside rules*: A rule that makes an API call for each account (to look up manager information) adds an API round-trip for every account. For 500,000 accounts, that's 500,000 API calls. Cache results in rule execution context or pre-process manager lookups as a separate operation.

*Complex regex in rules*: Regex evaluation is slower than string comparison. For simple cases (does this string start with "svc-"?), use `startsWith()` rather than regex.

*Unnecessary string operations*: Multiple `toLowerCase()`, `trim()`, and `substring()` calls on the same value add up. Store intermediate results in variables.

**Replace rules with built-in transforms where possible**: Built-in transforms are implemented in optimized Java code; BeanShell rules are interpreted. A built-in Lookup transform runs 10–100x faster than equivalent BeanShell logic.

### Identity Profile Optimization

ISC's identity profile applies transforms and evaluates rules for every identity after aggregation. Profiles with many complex rules slow down identity processing.

**Audit your identity profile transforms**: List every transform and rule applied in your identity profile. For each one, ask: is this still needed? Was it added for a use case that no longer exists? Unused transforms that still run waste processing time and add maintenance complexity.

**Reduce attribute propagation**: If the identity profile updates 40 attributes from multiple sources, each attribute update is processed and persisted. Reduce to the minimum attributes needed for governance decisions.

---

## Case Study: Reducing a 6-Hour Aggregation to Under 45 Minutes

**Environment**: Insurance company with 180,000 identities. Primary AD domain with full aggregation running nightly. Duration had grown from 90 minutes (2 years ago) to 6 hours (current).

**Diagnosis**:
- VA showing Degraded health during aggregation window (CPU at 95%+)
- ccg.log showed API response times from AD DCs averaging 800ms per page (vs 200ms 2 years ago)
- Identity profile had 34 transforms including 8 BeanShell rules
- 3 rules made external HTTP calls to a legacy employee lookup API
- Aggregation had grown from 120,000 to 180,000 accounts (50% growth — VA never resized)

**Changes made**:
1. VA CPU scaled from 4 to 8 vCPU, RAM from 16GB to 32GB
2. Added a second VA to the cluster (load sharing)
3. Preferred DC for ISC was over-loaded; AD team assigned a dedicated DC for ISC queries
4. 3 BeanShell rules using external HTTP calls replaced with a lookup table transform (pre-populated nightly from the same API via a separate process)
5. Delta aggregation implemented for the 6-hour window; full aggregation moved to Sundays
6. 12 identity profile transforms confirmed as unused and removed

**Results**:
- Delta aggregation: 6 minutes (replaces 6-hour nightly)
- Full aggregation (weekly): 43 minutes (down from 6 hours)
- Provisioning latency: reduced 40% (VA under less resource pressure)

---

## Provisioning Performance

Slow provisioning affects end-users directly — they request access, the provisioning takes 20 minutes instead of 2, and they're filing tickets with the help desk.

**Provisioning performance is usually a target system issue**: ISC submits the provisioning operation; the target system processes it. If Salesforce takes 8 seconds to respond to a user creation API call, ISC's provisioning will be slow regardless of ISC configuration.

**Parallel provisioning**: For roles that provision access across 6 target systems, ISC can provision all 6 in parallel rather than sequentially. Configure parallel workflow steps in the provisioning workflow. Sequential provisioning of 6 systems at 5 seconds each = 30 seconds. Parallel = 5 seconds.

**Provisioning queue depth**: If the provisioning queue has 500 items (large access certification remediation cycle), items at the back of the queue wait for items at the front. Process queues in priority order; configure maximum concurrent provisioning operations appropriately for VA capacity.

---

## Key Takeaways

- Identify the bottleneck phase before optimizing: retrieval (source-side), processing (transforms/rules), correlation, or persistence
- Full-to-delta migration is the highest-impact optimization for large sources — 200,000 accounts run as delta in minutes vs hours as full
- VA resource pressure is often undiagnosed — check CPU and memory during large aggregations; size up or add VAs when resources are saturated
- BeanShell rules with external API calls are high-cost at scale — cache results or replace with lookup transforms
- Dedicated domain controllers for ISC LDAP queries prevent source-system bottlenecks in AD environments
- Parallel provisioning (across multiple target systems simultaneously) reduces provisioning workflow duration proportionally

> 🔗 **CONNECTS TO:** Performance optimization addresses operational efficiency. Phase 8 moves to strategic level: planning large-scale ISC implementations, assessing organizational maturity, and building the expertise to lead complex projects.
