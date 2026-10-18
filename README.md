# cloud gpu comparison: real H100, A100, L40S and RTX 4090 hourly rates, the VRAM math tier lists get wrong, and when Vultr is the right pick

The same NVIDIA H100 can cost anywhere from roughly $1.50 to more than $12 per GPU-hour right now, depending on who you rent it from. That spread — close to 8x for an identical chip — is why the provider you pick matters almost as much as the card itself. On the card side, memory bandwidth and VRAM decide your real-world throughput and what actually fits, while raw FLOPS mostly decorate the spec sheet.

Any useful cloud GPU comparison has to work through both layers: what separates the GPUs you can rent today, what each class costs per hour, and where Vultr sits against marketplace platforms on one side and hyperscalers on the other.

> **Quick picks**
>
> - 7B–13B quantized models or image generation → an RTX 4090-class card, the cheapest option per token.
> - 70B in FP16 without sharding → a 141 GB card like the H200, or two 80 GB cards on NVLink. A single 80 GB H100 will not fit it.
> - Always-on workloads with fixed hourly rates and no marketplace roulette → [👉 check Vultr's current cloud GPU pricing](https://www.vultr.com/pricing/?ref=7268283) — L40S from $1.14/hr, A100 from $1.99/hr, H100 from $3.49/hr.

## What actually matters in a cloud GPU comparison

Memory bandwidth first, VRAM second, FLOPS a distant third. For LLM inference, how fast you can read model weights from memory predicts throughput better than raw compute does. That's why the H200 outperforms the H100 on inference despite essentially identical compute: its 4.8 TB/s of bandwidth feeds the tensor cores faster. Price trackers covering 5,000+ live instances across 18 providers currently put the H100 and RTX 4090 at the top of the price-to-performance rankings, and the specs below explain why.

| GPU | VRAM | Memory bandwidth | FP16 dense | FP8 dense | Interconnect |
| --- | --- | --- | --- | --- | --- |
| A100 80GB | 80 GB HBM2e | 2.0 TB/s | 312 | n/a | NVLink 3 |
| H100 SXM | 80 GB HBM3 | 3.35 TB/s | ~989 | ~1,979 | NVLink 4 |
| H200 SXM | 141 GB HBM3e | 4.8 TB/s | ~989 | ~1,979 | NVLink 4 |
| B200 | 192 GB HBM3e | 8.0 TB/s | ~2,250 | 4,500 | NVLink 5 |
| L40S | 48 GB GDDR6 | 864 GB/s | ~362 | ~733 | PCIe 4.0 |
| RTX 4090 | 24 GB GDDR6X | 1.0 TB/s | ~165 | ~660 | PCIe 4.0 |

*Tensor Core figures are dense (non-sparse). With NVIDIA's 2:4 structured sparsity, roughly double them: H100/H200 hit ~1,979 FP16 and ~3,958 FP8, L40S ~1,466 FP8. The A100 has no native FP8 support — FP8 arrived with Hopper and Ada Lovelace.*

Three rows deserve a second look.

**The L40S is the sleeper for inference.** At 48 GB with native FP8, it runs 13B models in FP16 with room for KV cache, and its FP8 throughput (733 dense, ~1,466 with sparsity) is far closer to an H100 than its price suggests. The trade-off is bandwidth: 864 GB/s means memory-bound work runs slower than on any HBM-equipped card.

**The A100 is aging out of on-demand pricing.** It's still a great card at $0.34/hr spot rates, but at typical on-demand rates the H100 costs barely more and runs roughly 3x faster in FP8.

**Avoid the museum pieces.** A T4 with 16 GB can't even hold a 7B model in FP16 — the weights alone need ~14 GB, leaving nothing for KV cache. The V100 survives in cloud catalogs mostly out of habit; an RTX 3090 at $0.07/hr beats it on every axis that matters.

## The VRAM math most tier lists get wrong

Work backwards from weight memory: parameters × 2 bytes in FP16, × 1 byte in FP8, × 0.5 bytes in 4-bit. Then leave 20–30% headroom for KV cache, activations, and framework overhead. That simple calculation settles most card choices before you ever open a pricing page.

- **7B–8B in FP16**: ~14 GB of weights → fits comfortably on a 24 GB RTX 4090.
- **13B in FP16**: ~26 GB → you need a 48 GB card like the L40S.
- **70B in FP16**: ~140 GB → no single 80 GB card can hold it, full stop.

That last line contradicts at least one popular tier list claiming a single H100 runs Llama 3 70B in FP16 "with room to spare for KV cache." The weights alone are ~140 GB; an 80 GB card can't hold them, KV cache or no KV cache. In practice you get there one of two ways. A 141 GB card like the H200 runs 70B in FP16 on one device with real cache headroom. Or you shard across two 80 GB cards — and that's where interconnect stops being a spec-sheet footnote. NVLink 4 on the H100 moves data between cards fast enough that sharding barely hurts, while two PCIe 4.0 cards (a pair of L40S or 4090s) will spend a lot of time waiting on each other.

## Vultr's cloud GPU lineup and current prices

Vultr rents NVIDIA GPUs as on-demand instances billed by the hour, at fixed rates: no bidding, no host trust scores, no guessing what hardware is on the other end. The current lineup covers the four cards most teams actually compare:

| GPU | VRAM | Memory bandwidth | Listed rate (per GPU-hr) | Deploy |
| --- | --- | --- | --- | --- |
| NVIDIA H100 | 80 GB | 3.35 TB/s | from $3.49 | [ Deploy the H100](https://www.vultr.com/products/cloud-gpu/?ref=7268283) |
| NVIDIA A100 | 80 GB | 2.0 TB/s | from $1.99 | [ Check A100 rates](https://www.vultr.com/products/cloud-gpu/?ref=7268283) |
| NVIDIA L40S | 48 GB | 864 GB/s | from $1.14 | [ Deploy the L40S](https://www.vultr.com/products/cloud-gpu/?ref=7268283) |
| NVIDIA RTX 4090 | 24 GB | 1.0 TB/s | from $0.73 | [ Check 4090 rates](https://www.vultr.com/products/cloud-gpu/?ref=7268283) |

Two caveats before you budget. Rates move as new hardware lands — H100 pricing has fallen through 2026 as Blackwell supply grows — so treat these as a snapshot and confirm live rates before committing. Vultr also keeps rotating newer SKUs (H200- and GH200-class) into its regions over time, so the live catalog is broader than any static table. [👉 Browse Vultr's full cloud GPU catalog](https://www.vultr.com/products/cloud-gpu/?ref=7268283) to see what's currently available where you deploy.

The four-card lineup maps cleanly onto the VRAM rules above: the RTX 4090 covers 7B–8B work, the L40S covers 13B-class FP16 plus FP8 batch inference, and the H100/A100 cover everything that needs 80 GB or NVLink.

## Vultr vs the neoclouds vs the hyperscalers

Provider tier swings the price of an identical GPU more than anything else on this page; specialized clouds typically run 40–85% cheaper than AWS, GCP, or Azure for the same silicon. Here's the H100 market as of late 2026, using published rates from each provider:

| Provider | H100 hourly | The catch |
| --- | --- | --- |
| Vast.ai | $1.49–2.21 marketplace range; dips under $1.60 on weekends | Reliability varies host to host; not built for production |
| RunPod | ~$1.50–1.99 depending on cloud and variant; $2.79 for Secure SXM | Community tier has no SLA; multi-node training is a known weak spot |
| Spheron | $2.75 on-demand / $2.20 spot per its own published rates | Spot-first model; suits fault-tolerant, checkpoint-heavy jobs |
| Lambda Labs | $2.49–3.99 SXM on-demand | Costs more; the value is InfiniBand clusters and a reservations desk for big runs |
| AWS | $12.29 per GPU-hr (p5.48xlarge at $98.32/hr ÷ 8 GPUs) | 4–5x neocloud pricing, plus egress and storage billed separately |

Read the top of that table carefully before crowning a "cheapest" winner. Much of the sub-$2 pricing is H100 PCIe on community or marketplace tiers, while SXM modules in certified facilities cost more everywhere. A $1.99 community PCIe card and a $2.79 Secure SXM card aren't the same product, and pretending they are is how comparison articles mislead people.

Where does Vultr land? Between the marketplace floor and the hyperscaler ceiling, deliberately. You pay more than a Vast.ai spot listing and less than an on-demand hyperscaler rate, and in exchange you get fixed infrastructure, hourly billing, availability across roughly 30 global regions, and GPUs that plug into the rest of a standard cloud stack — VPC networking, block storage, object storage, managed Kubernetes — without marketplace variability. For a production inference endpoint that needs to stay up, that trade is usually worth it. For a weekend experiment, it usually isn't.

If you'd rather not manage GPUs at all, per-token platforms like Together AI remove the infrastructure question entirely, pricing 200+ open models by request. Different product category, but worth knowing it exists.

## The cost math nobody runs: what a card costs per month

Multiply the hourly rate by 730 hours and the real bill appears. A card that sounds cheap by the hour is a serious line item by the month:

- **RTX 4090** — $0.73/hr → about **$533/month** running 24/7
- **L40S** — $1.14/hr → about **$832/month**
- **A100** — $1.99/hr → about **$1,453/month**
- **H100** — $3.49/hr → about **$2,548/month**

For comparison, a marketplace RTX 4090 at $0.39/hr runs about $285/month — but you're renting consumer hardware with no SLA, a fine trade for a hobby project and a bad one for anything customer-facing.

Two more things the hourly rate hides. First, spot pricing is usually cheaper, but verify per SKU: Spheron's own listings show the B300 spot rate ($5.84/hr) sitting above its on-demand rate ($5.40/hr), so "spot = discount" isn't a law of physics. Second, think twice before locking a long reservation. H100 prices dropped 47% in twelve months and kept falling as B200 and B300 supply grows; a two-year lock only makes sense if the discount versus today's rate is steep enough to survive further declines. Monthly commitments or plain on-demand keep your options open.

## When Vultr is the right call — and when it isn't

Pick Vultr when you want datacenter GPUs at predictable hourly rates, deployed in minutes, without gambling on marketplace hosts or paying the hyperscaler premium for enterprise features you don't use. It's the natural fit for always-on inference endpoints, dev and test environments that need to behave like production, and teams that want GPUs attached to normal cloud plumbing (networking, storage, Kubernetes) rather than an isolated GPU rental box.

It's the wrong pick in three specific cases. If your job is interruptible and cost is the only thing that matters, marketplace spot capacity undercuts it — pair that with frequent checkpointing. If you're running 4+ node training where interconnect topology decides whether gradient sync eats your throughput, Lambda's InfiniBand reservations are purpose-built for that and Vultr isn't. And if your traffic is bursty enough to benefit from scaling to zero, a serverless GPU product will bill you less than an idle instance. [👉 See Vultr's hourly GPU rates](https://www.vultr.com/pricing/?ref=7268283) if your workload falls on the "predictable and always-on" side of that line.

## Cloud GPU comparison FAQ

### Does FP8 support really matter when comparing GPUs?

Yes, and it's the quiet dividing line in every current lineup. The A100 has no native FP8 hardware; Hopper and Ada cards do. On models that support it, FP8 roughly doubles effective throughput — an H100 delivers ~1,979 dense FP8 TFLOPS versus ~989 in FP16 — which is why a newer card with FP8 can beat a bigger, older one that lacks it.

### Is a marketplace RTX 4090 the cheapest way to run small models?

For anything up to 13B quantized, yes. Market rates run $0.19/hr spot and $0.39/hr on-demand, and at those numbers the cost per token comfortably undercuts an A100 — a 4090 pushes Llama 3 8B quantized past 80 tokens/sec and generates SDXL images in about 1.2 seconds. The catch is reliability: check host trust scores, keep checkpoints, and don't put a customer-facing endpoint on someone's gaming rig.

### Should I lock in a long-term reservation now?

Only with a steep discount against today's rates. GPU prices have been falling all year as Blackwell supply grows, which means a reservation signed today is a bet that prices stop falling. They haven't yet. If your workload tolerates on-demand or monthly billing, stay flexible; commit only when the reserved rate beats the current on-demand rate by enough to survive another 20–30% market decline.

## The short version for your situation

- **Hobby projects and small models** → the cheapest marketplace 4090 spot you can find; if you prefer stable, fixed-billing hardware instead, Vultr's RTX 4090 at $0.73/hr is the same card without the lottery.
- **13B in FP16 or up to ~30B quantized** → the L40S, which punches well above its hourly rate on FP8 workloads.
- **70B in FP16** → an H200-class 141 GB card, or two 80 GB H100s on NVLink if you're sharding.
- **Large multi-node training** → InfiniBand-backed reservations rather than any single-GPU listing.
- **Always-on APIs with predictable bills** → [👉 spin up a Vultr GPU instance](https://bit.ly/VulTR) and match the card to the VRAM math above.

Rates on every platform in this piece move weekly. Whatever you pick, confirm the live price right before you deploy — the gap between checking and not checking currently runs about $10 per GPU-hour on the H100.
