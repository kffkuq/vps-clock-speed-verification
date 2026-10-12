# high frequency VPS: how to tell a real 5GHz+ machine from a marketing label, and what one actually costs

"High frequency VPS" sounds like a product category. It isn't one. No standards body defines it, no registry enforces it, and nothing stops a host from slapping the phrase on a 2.2GHz Xeon that has been doing the same job since 2019. What does exist is a genuine technical difference — machines that boost into the 5–6GHz range versus machines that sit at 2.2–2.6GHz — and a wide gap in what that difference costs you per month.

So the useful question isn't "which high frequency VPS is best." It's: **which workloads actually feel clock speed, how do you verify the clock you're paying for, and what should it cost?** Below is that, plus the current numbers from RainYun (雨云), a Chinese provider whose entire game-cloud line is built around exactly this kind of silicon.

## Where the term came from

"High frequency" as a hosting label traces back to Vultr's **High Frequency Compute** line, launched with a simple pitch: 3GHz+ Intel Xeon cores plus NVMe storage, sold alongside their cheaper regular cloud compute. The naming stuck because it named a real split. Standard budget VPS plans were running older, lower-clocked, more heavily oversubscribed CPUs, and "3GHz+" was a shorthand anyone could understand.

A decade of marketing later, the shorthand has drifted. Providers now use "high frequency" for:

- CPUs that idle at 2.4GHz and turbo to 3.1GHz for a few seconds under light load
- Shared vCPUs on a host where a dozen neighbours are doing the same thing
- Consumer desktop chips dropped into rented bare metal, which is genuinely a different animal

That last one is the interesting case, and it's where clock speed stops being a spec-sheet line and starts changing what your server can do.

## What clock speed actually buys you

The honest answer: nothing, for most sites. A WordPress blog, a static frontend, a Telegram bot, a lightly-used database — none of these care whether your vCPU runs at 2.5GHz or 5.8GHz. They're waiting on disk I/O, network round-trips, or simply nothing at all.

Clock speed matters when a **single thread** is the bottleneck and you can't parallelise your way out of it:

| Workload | Why one fast core wins |
| --- | --- |
| Modded Minecraft / game servers | The tick loop is single-threaded. More cores won't fix low TPS. |
| Trading bots, Solana RPC nodes, latency work | One process, one thread, and the clock speed is your headroom |
| Compiling, package builds, video encode with weak threading | Some steps are serial no matter what you do |
| Older PHP/Node apps, single-request latency testing | One request, one core, until you scale out |
| Emulation, single-player game hosting, physics sims | Same story |

If your workload is dozens of concurrent connections rather than one heavy process, buy cores and RAM instead. A 16-core machine at 2.4GHz will out-serve a 2-core machine at 5.8GHz for almost any web fleet.

## The part the spec pages leave out

Three things decide whether a "high frequency" plan is real, and none of them appear in the marketing copy.

**Turbo vs sustained.** Almost every modern server CPU has a boost clock it can't hold forever. A host quoting "3.8GHz" may mean a base clock of 2.6GHz with a burst. Ask for `lscpu` output or a single-core benchmark before you commit to a year.

**Shared vs dedicated vCPU.** A 5GHz core shared across eight tenants is a 5GHz core that spends most of its time busy. The number is true and the experience is still bad.

**P-cores vs E-cores.** This one is specific to Intel's 12th-gen and later desktop chips. Those CPUs have fast performance cores and slower efficiency cores, and schedulers sometimes park your workload on the slow ones. RainYun's documentation for its i9-14900KF and 13900KF machines explicitly states the chips are configured with **E-cores disabled**, meaning you're only ever scheduled on performance cores — which is a more meaningful statement than any clock number on a landing page.

## Verifying clock speed in about five minutes

Once you have SSH access to any VPS, three commands tell you more than the product page:


lscpu | grep -E "Model name|MHz"
cat /proc/cpuinfo | grep "MHz" | sort -r | head -3
curl -s https://raw.githubusercontent.com/... # or run a single-core bench


The first line gives you the CPU model, which you can look up for real base and boost clocks. Grepping `cpuinfo` under load shows what you're actually getting, not what the box idles at. Then run any single-core benchmark and compare it against published results for that CPU — a large gap means throttling, contention, or both.

For scale, published single-core results from RainYun's own benchmark posts and review coverage:

- **i9-14900KF** (Suqian, 4 vCPU, water-cooled, cited as overclocked to ~6.3GHz): Geekbench 6 single-core **2798**, multi-core 7914. CPU-Z single-core 907.
- **i9-14900K** (Zhenjiang, 4 vCPU): Geekbench 7 single-core **2362**, multi-core 7506.
- **EPYC 7452** (Frankfurt, 2 vCPU, 2.35GHz): Geekbench 6 single-core **1125**, multi-core 2001.

Same core count on the second and third rows — roughly **2.5x the single-core throughput**. That's what you're paying the premium for, and it's the number worth checking before you renew.

## What RainYun sells in this category

RainYun is a Chinese cloud provider that has been running since 2018 on its own KVM-based management platform. Two product lines matter here:

**Game cloud VPS (游戏云)** — the high-clock line. Documented CPU options span Intel **i9-14900K / 14900KF / 13900KF** (P-cores only, water-cooled, per RainYun's own notes), AMD **Ryzen 9950X** (Zen 5), **Ryzen 7950X**, **Ryzen 5900X**, plus older **Xeon Gold 6146** and **E5-2666v3** tiers at the budget end. Locations cover Chinese datacentres in Suqian, Zhenjiang, Ningbo, Wenzhou, Shaoxing and Chongqing, with 100–200Gbps shared DDoS protection on several of them.

**Cloud servers (云服务器)** — the general line, on Xeon Platinum/Gold, EPYC 7K62, EPYC 7702 and EPYC 7A23. Fewer exotic clocks, more location and bandwidth choice: Los Angeles, Tokyo, Hong Kong, Seoul, Frankfurt, Shenzhen, Hubei.

👉 [See RainYun's current high-clock configurations and live pricing](https://www.rainyun.com/MzA2MTc2_)

## The lineup, as listed

Preset game-cloud configurations with fixed monthly billing, per RainYun's published plan table (CNY, monthly):

| CPU | vCPU / RAM | Uplink | Disk | Listed ¥/month | Order |
| --- | --- | --- | --- | --- | --- |
| E5-2666v3 | 2 / 4GB | 10Mbps | 30GB + 20GB | ¥65 | Get this tier |
| Xeon Gold 6146 | 2 / 4GB | 10Mbps | 30GB + 20GB | ¥80 | Get this tier |
| Ryzen 5900X | 1 / 2GB | 5Mbps | 30GB + 20GB | ¥125 | Get this tier |
| Ryzen 7950X | 1 / 2GB | 5Mbps | 30GB + 20GB | ¥139 | Get this tier |
| Core i9-13900KF | 1 / 2GB | 5Mbps | 30GB + 20GB | ¥145 | Get this tier |
| Core i9-14900KF | 1 / 2GB | 5Mbps | 30GB + 20GB | ¥145 | Get this tier |

Those are raw list prices for preset shapes, and they make the trade-off blunt: an old 2-core E5 costs half what a single fast 5900X core costs. If you're picking preset small machines, the clock-speed premium is steep. Bigger hand-configured machines stack up better — a 4 vCPU / 8GB high-clock machine lists at **¥200/month** in Wenzhou and **¥220/month** in Zhenjiang (200Gbps defence), and those are the numbers most review coverage quotes.

Repeat purchases at annual billing take 30% off the monthly rate as standard. Multiple published guides describe stacking a 20% new-purchase coupon on top of that, landing near 44% off list — which is how a ¥200/month machine gets advertised at ¥112/month. Your actual figure depends on the coupon you're issued, so treat any "¥112" headline as a stacked-discount number, not a shelf price.

For readers who want an overseas box rather than a Chinese one:

| Region / platform | vCPU / RAM | Bandwidth | Listed ¥/month | Order |
| --- | --- | --- | --- | --- |
| Hong Kong zone 5 (EPYC 7K62) | 1 / 1GB | 200Mbps | ¥23 | Check this region |
| Hong Kong zone 5 | 2 / 4GB | 500Mbps | ¥43 | Check this region |
| Hong Kong zone 5 | 8 / 16GB | 1Gbps | ¥183 | Check this region |
| Seoul (EPYC 7A23) | 2 / 4GB | 40Mbps | ¥32.48 | Check this region |
| Seoul (EPYC 7A23) | 4 / 8GB | 75Mbps | ¥89.60 | Check this region |
| Frankfurt (Unicom-optimised) | 2 / 4GB | 100Mbps | ¥35.28 | Check this region |
| Frankfurt (Unicom-optimised) | 4 / 8GB | 200Mbps | ¥85.68 | Check this region |
| Los Angeles (EPYC 7702) | 2 / 2GB | 30M up / 50M down | ¥34 | Check this region |
| Los Angeles (EPYC 7702) | 8 / 16GB | 100M up / 150M down | ¥197 | Check this region |

Prices are in CNY because that's the currency the platform bills in. Overseas machines on this list are running EPYC chips in the 2.3–2.6GHz range — fast enough for web work, nowhere near the Chinese game-cloud machines if single-thread speed is the whole point. Pick according to the workload, not the location map.

## The billing model worth understanding

RainYun's game cloud supports two billing modes, and the second one is unusual enough to matter for anyone paying for clock speed they don't use 24/7.

- **Fixed billing** — the conventional monthly rent. Steady cost, full performance, no metering.
- **Dynamic billing** — a low base rent plus a daily charge based on CPU usage above a **20% baseline**. RainYun calls the unit CPU "electricity." Memory doesn't generate any.

The logic is aimed at game servers that sit idle between sessions: pay rent while idle, pay per-unit when players show up. If your high-frequency workload is a Minecraft server that's busy three evenings a week, dynamic billing is the cheaper shape. If it's a trading bot hammering one core around the clock, fixed billing is going to be less painful.

## Practical details that decide whether it works for you

**NAT vs dedicated IP.** Several of the cheaper high-clock machines are NAT instances — you share a public IP with other tenants and get a fixed number of port mappings (10 to 15 depending on machine and region). That's fine for a private game server and useless for hosting a site. Independent public IPs are purchasable as add-ons, and on high-defence machines the dedicated IP carries its own protection; figures quoted in review coverage run from ¥60/month for a Wenzhou line up to ¥150–¥200 elsewhere.

**Disk and bandwidth expansion are metered.** Additional SSD cloud disk is listed at ¥0.4/GB/month with an "efficient" tier at ¥0.1/GB/month. Extra bandwidth on game cloud machines has been quoted around ¥25/month per additional 5Mbps. Traffic-limited plans carry unused volume forward month to month, then throttle (often to 1–5Mbps) once exhausted, with top-up packages available.

**Try before you commit.** RainYun offers a ¥1 one-day trial per product (higher on expensive shapes) and a 7-day refund window, up to three times per account. Combined with the points mall — daily sign-in earns points, 2000 points converts to ¥1, and there's a free game cloud server you can redeem and renew with points — there's a low-cost path to finding out whether the silicon is what you were promised.

**Two caveats for non-Chinese buyers.** Purchases require real-name verification, and payment runs through Alipay or WeChat; the website and control panel are Chinese-first. If your Chinese reading is limited, budget time for translation, and check the current region list in the console before picking, because stock at specific machines comes and goes.

👉 [Start with a ¥1 one-day trial on RainYun](https://www.rainyun.com/MzA2MTc2_)

## Picking one, by workload

- **Modded Minecraft, Palworld, Terraria, single-instance game servers** — go straight for the i9-14900KF or Ryzen 9950X tiers. This is the case where the premium pays for itself, and RainYun's MCSM panel plus one-click deploy shortens setup. Pick a Chinese region if your players are there; the Wenzhou and Zhenjiang machines carry 100–200Gbps of shared defence.
- **A website, API, or anything with traffic** — skip the game cloud line entirely. The general cloud servers at ¥34–¥197/month in Los Angeles, or the Hong Kong zone 5 tiers from ¥23, give you dedicated IPs and real bandwidth, which is what a web workload actually needs.
- **Latency-sensitive bots or RPC nodes** — check the single-core benchmark first, then the network. A 6GHz chip behind a 150ms route loses to a 3GHz chip next door.
- **Testing what "high frequency" is worth to you** — buy one month of a cheap E5-2666v3 machine and one month of a 14900KF machine, run the same benchmark on both, and decide with your own numbers. That's two months of rent and it beats trusting anyone's adjective.

## FAQ

**Is a high frequency VPS the same as a dedicated CPU VPS?**
No. High frequency describes clock speed, which is usually shared between tenants on the same host. Dedicated CPU plans reserve cores for you, which addresses contention rather than clock speed — different problem, different price.

**Do I need a high-clock machine for a database?**
Usually not. Databases with concurrent queries scale on cores and I/O. You want clock speed when you have one heavy serial query and no way to index or shard your way out of it.

**Is a 3GHz+ claim enough to trust a provider?**
It's a starting point. Ask what the base clock is, whether cores are shared, and look for a single-core benchmark result from a machine in the region you plan to use. The gap between a claimed boost clock and a sustained benchmark is where disappointment lives.

**Can I run Windows?**
Yes — RainYun's cloud and game cloud products support both Windows and Linux, with Windows 2016 and later needed for things like Bedrock server binaries, and preinstalled stacks (Docker, Java, panels) available at deploy time.
