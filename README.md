# unmanaged dedicated server: full root control without the managed markup, plus DMIT plans and setup explained

You typed "unmanaged dedicated server" into a search box, which usually means one of a few things: you're tired of paying somebody else to babysit your server, you want a whole machine to yourself, or you're trying to figure out whether the cheaper unmanaged option is actually viable for whatever you're building. This article walks through what unmanaged really means in practice, where it saves you money, where it bites you, and how DMIT's bare metal and cloud plans fit into the picture — including the prices and specs that are actually on their site right now.

## What "unmanaged dedicated server" actually means

An unmanaged dedicated server is a physical machine rented to you alone, where the provider's responsibility stops at keeping the hardware powered on, connected to the network, and replaced if a component fails. Everything else — OS installation, kernel updates, firewall rules, SSH hardening, backups, patching, troubleshooting why nginx won't start at 3 AM — is on you.

That's the whole trade in one sentence. You get full root (or Administrator) access to a single-tenant box with no virtualization layer underneath, and in return you give up the hand-holding that "managed" plans bundle in. DMIT is explicit about this in their own Terms of Service, where they state that most of their services are unmanaged and they can only guarantee support ticket replies within 72 hours. So when you see "unmanaged" on a DMIT product page, that's not marketing language — it's the actual support posture.

The "dedicated" part matters too. A dedicated server (or bare metal server, the terms get used interchangeably) means the entire physical box is yours. No noisy neighbors on the same CPU, no vCPU oversubscription, no shared disk I/O. This is distinct from a VPS, which is a virtualized slice of a shared machine, and from managed hosting, where the provider handles the OS layer and often the application stack too.

## Where unmanaged dedicated servers fit compared to VPS and cloud

People often conflate these three, so let's be concrete.

A **VPS** is a virtual machine carved out of a hypervisor running on shared hardware. It's cheap, it's flexible, and for most small-to-medium workloads it's the right answer. The downside is that another tenant on the same host can hammer the disk or network and drag your performance down, even with CPU pinning and storage quotas in place.

A **cloud instance** is basically a VPS with a slicker API — snapshot, resize, autoscale, billed by the hour. DMIT's "Cloud Instance" product line is this category. It's still virtualized, still shared underneath, but you get self-service provisioning and modern tooling.

A **dedicated / bare metal server** is the actual machine. No hypervisor, no sharing. You get predictable performance because there's no contention, and you get hardware features that virtualization can't fully expose — direct NVMe controller access, specific CPU instruction sets, PCIe devices, the works. The trade is that you pay more, you wait longer for provisioning (or it's a custom quote), and you handle everything above the metal yourself.

If your workload is CPU-bound, latency-sensitive, or needs strict isolation for compliance reasons, dedicated wins. If you just need a box to run a web app and you want it to scale up on a Friday afternoon, cloud or VPS is usually more practical.

## The honest pros and cons

**What you gain:**

- Lower monthly cost than an equivalent managed dedicated plan, because you're not paying for someone's engineering time.
- Full control over the OS, kernel, and software stack. You can run a weird patched kernel, a custom init system, or a niche BSD variant without asking permission.
- Predictable performance. No neighbor-driven variance, no hypervisor scheduling jitter.
- Better isolation for sensitive data or regulated workloads where multi-tenant virtualization is a hard sell.

**What you take on:**

- You are the sysadmin. Security patches, firewall rules, log rotation, intrusion detection — all yours.
- If the box gets compromised, you clean it up. DMIT's TOS is clear that they're not liable for data loss and that you're responsible for your own backups.
- Support is limited. With DMIT specifically, the SLA on support ticket response is 72 hours, and that's for hardware/network issues — not "how do I configure nginx."
- Provisioning for true bare metal is usually a custom quote, not a one-click checkout. You wait.
- If you outgrow the box, you can't just drag a slider to add RAM. You migrate.

The unmanaged model only makes sense if you either have the skills in-house or you're willing to learn them fast. If your plan is "I'll figure it out when something breaks," you're going to have a bad time at 2 AM.

## DMIT's unmanaged server lineup: bare metal vs cloud instances

DMIT runs two relevant product lines, and it's worth being precise about which is which because they get conflated in search results.

**BareMetal Instance** is their true dedicated server product. Single-tenant physical hardware, full root and IPMI access, customizable CPU / RAM / disk, optional GPU, and a choice of network tiers (Premium CN2 GIA, Eyeball, or Tier 1). The bare metal page doesn't list fixed plans with prices — instead you describe your requirements and their team returns a tailored quote. That's standard for dedicated hardware but worth knowing before you go looking for a price table.

**Cloud Instance** is their virtualized product — AMD EPYC-backed VMs with full root access, free instant setup, snapshots, and automated backups. These are still unmanaged (same TOS clause applies), and they're what shows up on the public Pricing page with fixed monthly numbers. For a lot of people searching "unmanaged dedicated server," a high-spec cloud instance on dedicated-core hardware is functionally close enough, and it's the only DMIT product with publicly listed per-plan pricing.

Both lines run across the same three locations — Los Angeles, Hong Kong, and Tokyo — and the same three network series, so the routing story is consistent whether you go cloud or bare metal.

## Network tiers and why they matter

This is the part where DMIT diverges from a generic Hetzner or OVH pitch. They operate their own network with direct peering into China, and the tier you pick changes both price and performance meaningfully.

**Premium Network** uses China Telecom CN2 GIA plus DMIT's own backbone. This is the expensive tier and the one to pick if your users are in mainland China or APAC and you care about latency and packet loss. DMIT publishes ~15ms average latency to China Mainland from Hong Kong with packet loss under 0.1%, which is dramatically better than standard Tier 1 transit into China.

**Eyeball Network** pairs Tier 1 transit with reasonable-effort China routing via CMI and other Chinese eyeball ISPs. It's the middle ground — better for Chinese residential users than plain Tier 1, cheaper than Premium, but without routing guarantees. Good for mixed global/China audiences.

**Tier 1 Network** is the budget option: clean routing across APAC and the Americas via multi-Tbps Tier 1 backbones, no China-specific optimization. Use this for backup servers, bulk transfers, internal tooling, VPN relays, or anything where China latency isn't a factor.

The same workload can cost 3-4x more on Premium versus Tier 1, so picking the wrong tier is an expensive mistake. If your users aren't in China, Tier 1 is almost always the right call.

## Hardware platforms by location

DMIT runs multiple AMD EPYC generations depending on location, and the platform affects single-core performance noticeably.

In **Los Angeles**, they offer three platforms:
- **AN5 (AMD EPYC 9005 / Zen 5)** — flagship, DDR5, PCIe 5.0 NVMe. Best single-core performance.
- **AN4 (AMD EPYC 9004 / Zen 4)** — balanced, field-tested, the workhorse for most plans.
- **AS3 (AMD EPYC 7003 / Zen 3)** — best price-per-core, but the LAX AS3 platform is still being built out and DMIT warns of reduced disk performance and lower SLA during this period.

In **Hong Kong**, they offer AN5 (Zen 5) and AS3 (Zen 3) platforms. AN5 plans in Hong Kong are Premium-network only.

Tokyo details weren't fully enumerable from the public pages, but it's the third location in their triad and follows the same network series structure.

If you're benchmark-sensitive, pick AN5. If you're budget-sensitive and don't mind a known-immature platform note on LAX AS3, AS3 is the value play.

## DMIT cloud instance pricing — what's actually on the site

The prices below are pulled from DMIT's public Pricing page for the Los Angeles Premium Network / AS3 series configuration. DMIT notes that products and prices may not always be updated in real time, so treat these as reference and confirm on the page before checkout. Hong Kong AN5 plans are priced higher and listed separately.

| Plan | vCores | RAM | SSD | Transfer | Port | Price (USD/mo) | Get it |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TINY | 1 | 2GB | 20GB | 1000GB | 1Gbps | $10.90 | [View on DMIT](https://bit.ly/DmiT) |
| Pocket | 2 | 2GB | 40GB | 1500GB | 4Gbps | $16.90 | [View on DMIT](https://bit.ly/DmiT) |
| STARTER | 2 | 2GB | 80GB | 3000GB | 10Gbps | $34.90 | [View on DMIT](https://bit.ly/DmiT) |
| MINI | 4 | 4GB | 80GB | 5000GB | 10Gbps | $62.90 | [View on DMIT](https://bit.ly/DmiT) |
| MICRO | 4 | 4GB | 160GB | 7000GB | 10Gbps | $87.90 | [View on DMIT](https://bit.ly/DmiT) |
| MEDIUM | 6 | 8GB | 160GB | 15000GB | 10Gbps | $199.90 | [View on DMIT](https://bit.ly/DmiT) |

These are the LAX Premium / AS3 reference prices. Switching to a different network series (Eyeball, Tier 1) or a different platform (AN4, AN5) changes the number, and switching to Hong Kong AN5 Premium changes it a lot more — the Hong Kong AN5 lineup starts at $149.90/mo for the MINI plan (4 vCore / 4GB / 80GB / 1500GB transfer / 1Gbps) and runs up to $759.90/mo for the GIANT plan (12 vCore / 24GB / 640GB / 6000GB transfer). The Hong Kong premium reflects CN2 GIA capacity, which is a finite and expensive resource — that's the trade for ~15ms China latency.

For the bare metal dedicated servers, pricing is by custom quote. You describe the CPU, RAM, storage, bandwidth, and IP requirements, and DMIT returns a tailored proposal. There's no public per-plan price table to list here, and I'm not going to invent one. If you want a true single-tenant box, 👉 [reach out to DMIT for a bare metal quote](https://bit.ly/DmiT) and tell them your workload.

## What you're actually responsible for on an unmanaged DMIT server

Once the box is up, here's the non-exhaustive list of things DMIT will not do for you:

- **OS installation and reinstalls.** Cloud instances have a self-service reinstall panel, but choosing the OS image and post-install configuration is on you.
- **Security hardening.** SSH key-only auth, firewall rules, fail2ban or equivalent, disabling unused services, keeping the kernel patched.
- **Backups.** DMIT's TOS states explicitly that they're not liable for data loss and that you're solely responsible for creating backups. They offer snapshots and automated backups on cloud instances as a feature, but routine backup strategy is yours to design and test.
- **Application stack.** Installing and maintaining nginx, Postgres, Docker, whatever you run.
- **Monitoring and alerting.** If the box goes down at 4 AM, you find out because your monitoring told you — not because DMIT paged you.
- **DDoS response beyond mitigation.** DMIT does have DDoS protection on their network, but note their refund policy specifically excludes DDoS-targeted services from refund eligibility, which tells you something about how they view attack-related disruption.

DMIT's SLA, per their TOS, is currently 99%. If SLA drops below 99% you get half a month's credit; below 95%, a full month; below 90%, two months. That's a network/power SLA — not a "your app works" SLA.

## How to decide: unmanaged dedicated, unmanaged cloud, or managed

A rough decision flow:

1. **Are your users in mainland China or APAC, and does latency matter?** Strongly consider DMIT's Premium tier, and strongly consider Hong Kong over LAX. The CN2 GIA routing is the actual reason to pick DMIT over Hetzner or OVH in the first place.
2. **Do you need a whole physical machine, or is dedicated-core virtualization enough?** If a 6-12 vCore cloud instance on AMD EPYC dedicated cores handles your load, you don't need bare metal. Save the money and skip the custom-quote wait. If you're doing CPU-bound database work, rendering, or compliance-driven isolation, go bare metal.
3. **Can you actually admin a Linux box?** Be honest. If the answer is "no, but I'll learn," budget time for it — a couple of weekends to get comfortable with systemd, iptables/nftables, SSH hardening, and a backup routine. If the answer is "no and I don't want to," you want a managed plan, not an unmanaged dedicated server, and DMIT is the wrong vendor for that.
4. **What's your failure mode if the box dies at 3 AM?** If "I restore from backup onto a new instance within an hour" is your plan, cloud instances with snapshots work. If "I wait for DMIT remote hands to swap a failed drive" is the plan, that's bare metal, and the recovery time is longer.

The unmanaged dedicated server pitch is not "it's better." It's "it's cheaper and more powerful if you can carry the operational load yourself." If you can't, the savings evaporate the first time something breaks and you don't know how to fix it.

## Setting up an unmanaged server: the short version

If you're going ahead, here's the rough sequence most people follow on a fresh DMIT box. This isn't a full tutorial — it's a reality check on what's involved.

1. **Pick location and network tier.** LAX Tier 1 for budget global workloads, LAX Premium for China-facing, Hong Kong Premium for the best China latency, Tokyo for APAC generally.
2. **Provision.** Cloud instances are instant via the panel. Bare metal is a custom quote and a longer wait.
3. **First boot.** Log in via SSH with the credentials or key the panel gives you. Change the root password immediately if password auth was enabled.
4. **Harden SSH.** Disable password auth, disable root login in favor of a sudo user, move the port if you want fewer drive-by scans.
5. **Firewall.** Set up nftables or ufw. Default deny inbound, allow only what you need (22 or your SSH port, 80/443 if it's a web server).
6. **Updates.** Patch the base system. Set up unattended-upgrades or a scheduled patching routine.
7. **Fail2ban or equivalent.** Brute-force protection on SSH and any exposed auth endpoints.
8. **Backups.** Define what gets backed up, where it goes (off-box, ideally off-provider), how often, and test a restore. DMIT snapshots are not a backup strategy on their own — they're stored in the same infrastructure as the instance.
9. **Monitoring.** At minimum, uptime monitoring from an external service. Better: CPU, disk, memory, and key-process alerting via Prometheus + Grafana, or a hosted equivalent.
10. **Application stack.** Now you actually install the thing you wanted the server for.

If step 4 through 9 read like a foreign language, that's the signal to reconsider whether unmanaged is the right model for you right now. It's learnable, but it's not free.

## Things DMIT does that most generic providers don't

A few specific things came out of the research that are worth flagging because they're not standard across the bare-metal field:

- **Direct peering with all three major Chinese carriers** (China Telecom AS4809, China Unicom AS9929, China Mobile International AS58807). Most US/EU providers rely on transit to reach China, which is where the packet loss and jitter come from. DMIT's direct peering is the core differentiator.
- **CN2 GIA as a first-class network tier.** Most providers treat China-optimized routing as a paid add-on or don't offer it at all. DMIT builds it into the Premium network series.
- **Tri-location Pacific Rim footprint** (LAX, HKG, TYO) all on the same network backbone, which makes multi-region APAC deployments cleaner than stitching together providers.
- **BGP and BYOIP support on bare metal.** You can announce your own IP space, which is unusual outside of enterprise-focused providers.
- **Refund policy is narrow.** Full refund only within 3 days and under 30GB transfer used; partial refund up to 30 days with transfer-based calculation; no refunds on renewals, credit-funded orders, DDoS-targeted services, or "network not good enough" claims. Read the TOS before you buy, not after.

That last point is a real consideration. DMIT's refund window is tighter than the industry norm, which is fine if you've done your homework but painful if you buy on impulse and find the route to your users isn't what you hoped.

## Common questions

**Is "bare metal" the same as "dedicated server"?** In practice, yes. Both mean a single-tenant physical machine. Some vendors use "bare metal" to emphasize no virtualization layer and "dedicated server" to emphasize a managed-style product wrapper, but the underlying hardware model is the same. DMIT uses "BareMetal Instance" as their product name.

**Can I get a refund if the China routing isn't good enough for my users?** Per DMIT's TOS, "the network is not good enough" is explicitly listed as a non-refundable case. The refund policy is also void if you've used more than 3GB transfer and the issue is that the IP isn't reachable in some region. The right move is to test routing before committing to a long billing period — start monthly, not annual.

**Does DMIT offer managed services?** Their TOS says most services are unmanaged, with a 72-hour ticket response guarantee. They don't market a managed dedicated product on the public pages. If you need managed, look elsewhere.

**Why are Hong Kong plans so much more expensive than Los Angeles plans?** CN2 GIA capacity into Hong Kong is a finite, high-cost resource, and Hong Kong real estate and power are expensive. You're paying for the ~15ms China latency and the low packet loss. If your users aren't in China, there's no reason to pay for it.

**Can I run Windows on these?** DMIT's cloud instances support various OS templates via their reinstall panel; check the available templates at provisioning time for current Windows support and licensing. For bare metal, OS choice is part of the spec conversation with their team.

**What's the actual support experience like?** The TOS promises 72-hour ticket response, and community discussion of bare metal providers generally — not DMIT specifically — flags that unmanaged means you should not expect help with anything above the hardware/network layer. Plan accordingly.

## Bottom line

An unmanaged dedicated server is the right call when you have the sysadmin skills (or the willingness to acquire them), a workload that actually benefits from single-tenant hardware, and a clear picture of which network tier your users need. It's the wrong call if you're chasing the cheapest possible price tag without budgeting the operational cost of running the box yourself.

DMIT specifically makes sense when China or APAC latency is part of your decision — that's where their network investment actually shows up, and it's the reason to pick them over a cheaper generic provider. If your users are all in Europe or the US east coast, you're paying for routing you don't need, and a Hetzner or OVH box will do the same job for less.

If you've read this far and the unmanaged model still fits, the next step is picking a tier and a location. 👉 [Browse DMIT's current cloud instance plans and pricing](https://bit.ly/DmiT) for the virtualized option with public pricing, or if you want a true single-tenant box, 👉 [request a custom bare metal quote](https://bit.ly/DmiT) with your workload spec. Either way, start on a monthly billing cycle until you've verified the route to your actual users — the refund window is short enough that you don't want to find out there's a problem three months into a prepaid year.
