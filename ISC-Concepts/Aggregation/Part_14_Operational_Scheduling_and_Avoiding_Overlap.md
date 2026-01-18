# Part 14 – Operational Scheduling and Avoiding Overlap — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**How do I make time work *with* the engine instead of against it?**

Scheduling is not picking a clock.  
Scheduling is deciding how much truth the engine must carry at once.

Keep this in mind:

**Bad schedules don’t just run late. They change outcomes.**

---

## Where This Lives in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Scheduling controls the Trigger, but it shapes everything after it.  
It decides how often truth is refreshed and how much pressure the engine feels.

---

## Mental Model

```
Time choice
   → Job creation rhythm
     → Worker load
       → Delta memory stability
         → Downstream recompute pressure
```

If time is chaotic, everything after it becomes chaotic — even if logic is perfect.

---

## A Story: Two Teams, Same System

Team A schedules everything at midnight.  
Team B staggers sources with purpose.

At 12:00 AM:
- Team A launches 12 jobs
- Queue fills
- Some jobs overlap
- Delta memory collides
- Recompute storms begin

Team B:
- HR at 4:30 AM
- AD at 5:30 AM
- Apps spread through the day

By 8:00 AM:
Team B has fresh identities and calm access.  
Team A is still waiting for yesterday to finish.

Same engine.  
Different time decisions.

---

## What Scheduling Really Controls

People think scheduling controls *when*.

It actually controls:

- How many jobs compete at once
- How often delta memory changes
- How much downstream recompute happens
- How stale identity can become

Time is a load balancer.

---

## The First Truth: Duration Beats Frequency

Before you choose “daily” or “hourly,” ask:

- How long does this job take on a bad day?
- How slow can the source get?
- How long does recompute take after?

If a job can take 45 minutes,  
scheduling it every 30 minutes guarantees overlap — even if you don’t see it immediately.

Overlap is not a maybe.  
It is math.

---

## Overlap: Why It Is So Dangerous

When two jobs overlap on the same source:

- They fight over workers
- They may confuse delta memory
- They may write in unpredictable order
- They may hide each other’s failures

The engine assumes one storyteller at a time.  
Overlap creates two narrators talking over each other.

---

## The Second Truth: Order Matters

Not all sources are equal.

Some sources create people.  
Others only decorate them.

Authoritative sources (like HR) define:
- Who exists
- Lifecycle state

Non-authoritative sources (like AD, apps) attach access.

So order must be:

People first → Access later

If access runs before people exist,  
correlation suffers and cleanup follows.

---

## A Slow Walk Through a Good Order

HR runs at 5:00 AM.  
It creates and updates identities.

AD runs at 6:00 AM.  
It finds identities ready and correlates cleanly.

Apps run later.  
They attach access to already-known people.

The engine flows like a river, not a traffic jam.

---

## Bursts: The Hidden Enemy

A burst is when many jobs start together.

Bursts cause:

- Long queues
- Worker starvation
- Slow delta responses
- Recompute waves

It feels efficient to “run everything at midnight.”  
It is not. It is noisy.

Think traffic lights, not drag racing.

---

## Delta and Time

Delta is memory-based.

If you change memory too often:

- Tokens churn
- Risk of silent skips rises
- History becomes fragile

Delta likes calm rhythms.

Healthy pattern:

- Delta regularly
- Full occasionally

Example:
Delta daily.  
Full weekly.

Not because full is faster —  
because memory needs periodic grounding.

---

## Downstream Is Real Work

Aggregation does not end when the job ends.

After each run:
- Identities may re-evaluate
- Access may recompute
- Indexing may refresh

If you run heavy jobs back-to-back,  
you stack waves of judgment on top of each other.

This looks like:
- Access arriving late
- UI feeling frozen
- Queues that never empty

The engine is not broken.  
It is exhausted.

---

## Interactive Pause

You schedule HR every 15 minutes.

On bad days, HR job takes 25 minutes.

Question:
What will eventually happen?

Pause. Think.

Answer:
Jobs will overlap, delta memory will compete, and outcomes will become unpredictable — even if jobs still say “Completed.”

---

## Scheduling for Business Reality

Time is chosen for humans, not machines.

Ask:
- When do joiners need access?
- How fast must leavers lose access?
- When are admins awake to watch failures?

Then place authoritative sources early.  
Place access sources after.  
Place low-impact systems later.

Schedule for people, not just processors.

---

## Why Scheduling Creates Illusions

You may see:

- Jobs say Completed
- UI shows green
- But identity is stale

That often means:
Your schedule starved the engine of calm time to finish judgment.

Time lied to you politely.

---

## Traps That Fool Smart People

- Scheduling too frequently “just to be safe”
- Running everything at the same time
- Ignoring job duration trends
- Forgetting downstream recompute cost

These are not beginner mistakes.  
They are rushed-operator mistakes.

---

## Debug Mindset for Scheduling

When timing feels wrong, ask:

1) How long does each job really take?
2) Do any jobs overlap?
3) Are bursts happening?
4) Is recompute piling up?
5) Is delta memory stable?

Fix rhythm before fixing logic.

---

## Visual Rhythm Check

```
Job duration
   vs
Schedule gap
   ↓
Is gap always larger?
   ↓
Are jobs staggered?
   ↓
Is downstream calm?
```

---

## What This Phase Does NOT Do

- It does not change data
- It does not fix mapping
- It does not fix correlation

It only controls *how calmly* truth is allowed to move.

---

## Safe Scheduling Principles

- Never schedule sooner than worst-case duration
- Run authoritative sources first
- Stagger, don’t burst
- Pair delta with periodic full
- Leave time for downstream judgment

---

## The One Sentence That Defines Mastery

Before you choose a time, ask:

**Will this rhythm keep the engine calm when it is tired?**

---

## Mastery Check

Answer without notes:

- Why is overlap more dangerous than slowness?
- Why must people-creating sources run first?
- Why do bursts cause recompute storms?
- Why does delta need calm memory?
- Why is time a load balancer?

---

### Navigation
⬅️ Previous: [Part 13 – Proof Paths](./Part_13_Proof_Paths_and_Fix_Playbooks.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 15 – Custom Scheduling](./Part_15_Custom_Scheduling_Patterns_Weekends_Windows_and_Exceptions.md)

