+++
title = "Matrix isn't highly available, and that's okay"
date = "2026-08-08"
authors = ["Logan Devine"]

[taxonomies]
tags = ["matrix", "infra"]
+++

As part of my ongoing effort to make my infrastructure more reliable and learn more about how to
design reliable systems, one of the most important things I host that would benefit from reliability
would be my Matrix stack. There's one big problem: Matrix isn't designed for high availability.
You can't just deploy two Synapse servers, set them behind a reverse proxy, and expect it to work.

So, why should someone bother with high availability anyway, especially for a 4-user homeserver
like zirco.dev?

# Designing for failure

Computers fail. They always have, and that will never change during my lifetime. Just like the
classic saying with backups: it is not a question of _if_ a hard drive will fail, it's a question of
_when_. The same is true for all hardware, software, and networks. If you're running a service that
needs to be available, you need to design for failure.

When it comes to computer infrastructure, the most common form of fail-proof design is simple
redundancy. If one web server explodes, you configure a second, alongside a load balancer to route
traffic to whichever is healthy.

<!-- prettier-ignore-start -->
{% mermaid() %}
flowchart LR
    A[User] --> B[Load Balancer]
    B --> C[Web Server 1]
    B --> D[Web Server 2]

    style A fill:#f9f,stroke:#333
    style B fill:#9f9,stroke:#333
    style C fill:#99f,stroke:#333
    style D fill:#99f,stroke:#333
    linkStyle default stroke:#ef5350,stroke-width:2px
{% end %}
<!-- prettier-ignore-end -->

This scales well with stateless services (such as my `www-static` nginx cluster), but when you start
to introduce API servers, databases, and other stateful services, things get a lot more complicated.
One needs to worry about data replication, a [consistency model](https://en.wikipedia.org/wiki/CAP_theorem),
and how to handle failover or load balancing.

This architecture is also known as "active-active" (or "active-passive" if the second server is
only a hot standby) and provides not just high availability, but also horizontal scalability. This
logic, however, does not apply to the Matrix homeservers of today's world.

# Matrix isn't highly available

Matrix is a demanding protocol, and most servers consist of some form of _horizontal scaling_:

| Implementation | Horizontal scaling | Notes                                                                                                       |
| -------------- | ------------------ | ----------------------------------------------------------------------------------------------------------- |
| Synapse        | Worker processes   | Each worker can be delegated a specific task, but the master still performs many core functions.            |
| Dendrite       | Polylithic         | Each service can scale independently, but the services are tightly coupled and not designed for clustering. |
| Continuwuity   | Threaded           | Threads are used (async Rust), but there is still only one process.                                         |

However, notice that none of these implementations are designed for clustering. Although it would
be possible, keeping multiple masters in sync would be a nightmare.

Matrix servers are highly stateful: they keep track of room state, keep extensive caches, and
perform state resoluton. Given enough research into proper replication within a server, who knows,
it may happen one day.

> Please do not curse me with IRC-style netsplits on Matrix...

{% note(header="A note on hypervisor-level HA features") %}

Many hypervisors (such as Proxmox and `corosync`) feature native high availability for full node
failures. We do not utilize this functionality yet, and although it is a viable option, it is not a
complete solution and does not address all possible failures. Additionally, I am primarily
interested in building redundancy into the software stack, without relying on ring -1 to do it for
me.

{% end %}

# Who cares?

> I pay for whole server, I use whole server.

Given the opportunity, I believe that everyone should take the time to figure out their infrastructure
in such a way where it can be online more. It helps all of us, especially in a decentralized world,
but it also helps yourself. Building reliable systems is a useful lesson for everyone, as it helps
you not just prepare for crisis but also helps you design systems that are easier to maintain.

Plus, who doesn't want to flex uptime stats?

# Redefining "high availability"

> High availability (HA) is a characteristic of a system that aims to ensure an agreed level of
> operational performance, usually uptime, for a higher than normal period.  
> \- <cite>[Wikipedia](https://en.wikipedia.org/wiki/High_availability)</cite>

Per the above definition, high availability is the overall characteristic of being available. It
does not mandate any specific methodology.

If one cannot make a service itself redundant, the system around it can be designed such that all
reasonable failure modes are accounted for and have as small of an impact as possible. For me, that
means:

- **Observability:** I need to know when something is wrong, and metrics, logs, and alerting should
  help me identify the root cause quickly.
- **Fast recovery (low RTO):** All systems should be designed such that restoration to a working
  state (whether that be a snapshot restore, a rebuild, or a failover) is as fast as possible.
    - **RTO (Recovery Time Objective)** is the maximum acceptable length of time that a system can
      be unavailable after a failure.
- **Low RPO (Recovery Point Objective):** All systems should be designed such that the amount of
  data lost in the event of a failure is minimized. This can be achieved through frequent backups
  or live replication.
- **Redundancy where possible:** If a service can be made redundant, it should be. This includes on
  a hardware level (RAID, dual power supplies, etc.) and on a software level.
- **Tested recovery:** All of this is useless if not tested.

## Accepting the single point of failure

# Observability first

# Building the infrastructure

# Testing for failure

# Where to go from here
