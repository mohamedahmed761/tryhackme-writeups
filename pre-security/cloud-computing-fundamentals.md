# Cloud Computing Fundamentals

**Path:** Pre-Security → Computer Fundamentals
**Room:** [Cloud Computing Fundamentals](https://tryhackme.com/room/cloudcomputingfundamentals)
**Difficulty:** Easy · **Time:** ~30 min · **Completed:** 2026-05-22

## What the room covered

Why cloud exists, its benefits, deployment types, service models, the major vendors, and a hands-on console simulator where you deploy EC2 instances and watch the bill change.

Tasks: intro, Cloud Computing Overview (concepts), Deploying a Cloud Instance (hands-on console), conclusion.

## Cloud concepts

**Why cloud.** Move off one physical box to resources rented over the internet — easier access, more reliable, scales with demand.

**Benefits.**
- Scalability — scale up/down as needs change.
- On-demand self-service — spin up servers/storage instantly, no hardware wait.
- Pay for what you use — usage-based, no big upfront cost.
- Security — provider hardens the infrastructure.
- High availability — keeps running if part of the system fails.
- Global access — reachable from anywhere.

**Deployment types.**

| Type | Who uses it | Why |
| --- | --- | --- |
| Public | startups, websites, global apps | cheap, easy to scale, no infra to manage |
| Private | banks, healthcare, government | control and compliance for sensitive data |
| Hybrid | e-commerce | keep sensitive data private, scale publicly on spikes |

**Service models.**

| Model | You manage | Provider manages | Example |
| --- | --- | --- | --- |
| IaaS | OS + your app | physical hardware | rent a virtual server |
| PaaS | just your app | infra + OS | app hosting platforms |
| SaaS | nothing, just use it | everything | Gmail, Zoom |

Rent-a-home analogy: IaaS = empty apartment you furnish; PaaS = furnished place; SaaS = hotel room.

**Vendors.** AWS (leader), Azure (enterprise/hybrid), GCP (data/AI/ML), Alibaba (Asia), IBM (hybrid/AI), Oracle (enterprise apps/databases).

**Real users.** Netflix, Spotify, Instagram, online stores on Black Friday — all lean on cloud to scale, stay reliable, and skip owning hardware.

## Hands-on — deploying instances

Used a simulated AWS-style console (Task 3, "View Site"). IaaS model, since security work needs full OS access.

Terms:
- **EC2** = a virtual computer (CPU + RAM) in the cloud.
- **Instance type** (t3, m5, ...) = how powerful it is. Bigger = more power + higher cost.

Steps:
1. Pick a region (geographic location, top-right).
2. Create the app interface machine — `application-interface`, `t3.micro`, running.
3. Create two study machines — `study-machine-1` and `study-machine-2`, both `m5.large`, running.
4. Check Billing — see per-instance and total cost.
5. Stop both study machines, recheck Billing — cost drops sharply.

Point of the exercise: in the cloud you create, resize, and stop resources in seconds, and the bill follows usage.

## What clicked

The billing section. Spinning up three machines, then stopping two and watching the cost drop, made "pay only for what you use" real — it's not a slogan, the number actually moves.

## What to revisit

Service models — IaaS vs PaaS vs SaaS. The rent-a-home analogy helps, but I want to pick the right one for a given scenario without thinking.
