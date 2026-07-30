# PoC Request Email Template

> An email **you send to the TiDB / PingCAP team** to open or move along a PoC. It packages
> what you've worked out here — your environment, the offering that fits, the migration
> approach, and the questions still open — so the first conversation starts from substance
> instead of a blank questionnaire.
>
> Fill this in from the assessment report and send it. The report goes attached.
>
> Rules for filling it in:
> - **Lead with your constraints, not your questions.** The constraints are what let them
>   answer usefully in one round instead of three.
> - **Say what you've already concluded, and why.** If they disagree they'll say so, and that
>   disagreement is useful — better than them re-deriving everything from scratch.
> - **Keep open questions specific and numbered.** Vague asks get vague answers.
> - **State your deadline plainly** if you have one. It changes how they sequence things.
> - **Don't leave out the awkward parts** — a feature you rely on that isn't supported, an
>   internal approval you haven't started. Surfacing them now is what makes the timeline real.

---

**Subject**: TiDB Cloud PoC request — {{your company}}, {{source database}} migration

Hi,

We're evaluating TiDB Cloud for {{one line on the system — e.g. "our multi-tenant ERP
platform"}} and would like to run a PoC. I've put our environment and requirements below so we
can skip the discovery round; the full assessment is attached.

**Our current environment**

- **Source**: {{database and version, e.g. Google Cloud SQL for MySQL 8.0}}
- **Data volume**: {{raw size}}{{, largest table if known}}
- **Throughput**: {{peak QPS/TPS}}, roughly {{read:write ratio}}
- **Latency target**: {{P95/P99 target, or "no formal SLA"}}
- **Cloud / region**: {{provider and region}}
- **Workload**: {{transactional / transactional + reporting / analytical}}

**What we need**

- {{Hard requirement, e.g. "VPC peering — security and performance"}}
- {{Hard requirement, e.g. "continuous replication from production; we can only take N minutes
  of cutover downtime"}}
- {{Anything else constraining the design — compliance, data residency, tenant isolation}}

**Where we landed, and why**

Based on the above, **{{offering}}** looks like the fit — {{the reasoning in one or two
sentences, naming the constraints that drove it rather than the features that attracted us}}.

{{If two or more requirements independently point to the same offering, say so — it means
there's little to debate and you can go straight to sizing.}}

{{If preview or beta features are involved, name them and say what you need to know: how
stable they are, what the support position is, and whether they suit your timeline.}}

**On the migration**

{{Two or three sentences: the approach you expect (bulk load, continuous replication, or
both), the cutover downtime you're targeting, and anything you've already verified or
started.}}

**Open questions**

1. {{Specific question}}
2. {{Specific question}}
3. {{Specific question}}

**Timeline**

{{Your target window and what's driving it. Note what's already moving on your side — billing
setup, network changes — and what you'd need help with.}}

Happy to get on a call. Who's the right person to work with on this?

Thanks,
{{your name}}
{{role, company}}

---

## Variant — when a requirement conflict came up

Replace "Where we landed, and why" with:

> **A conflict we need your input on**
>
> Two of our requirements appear incompatible: {{requirement A}} points to {{offering X}}, but
> {{requirement B}} rules it out. From the documentation we couldn't find a single offering
> covering both.
>
> {{If it involves vector or full-text search: The docs are also inconsistent on which tiers
> support it — the feature comparison page and the feature's own overview page disagree — so
> before we redesign anything, could you confirm the actual availability?}}
>
> If the constraint is real, the options we can see are:
>
> 1. **{{Route A}}** — {{one line}}. Our concern: {{cost / complexity / risk}}.
> 2. **{{Route B}}** — {{one line}}. Our concern: {{cost / complexity / risk}}.
> 3. **{{Route C}}** — {{one line}}. Our concern: {{cost / complexity / risk}}.
>
> Given {{your priority}}, we'd lean toward {{route}} — but we'd rather hear whether there's an
> option we've missed before committing to an architecture.

---

## Variant — when your source database isn't MySQL-compatible

Add after "Our current environment":

> Coming from {{source}}, we know the schema and data conversion needs more than a
> protocol-compatibility check — {{the specific concern, e.g. "we have PL/SQL packages" or
> "our type mappings won't be one-to-one"}}. We'd want that assessed properly rather than
> assumed, so tell us what you need from us to scope it.

---

## Variant — when you have unsupported features to deal with

Add before "Open questions":

> **Something we'll need to solve together**
>
> Our schema relies on {{feature}}, which as far as we can tell TiDB doesn't support. Our
> current thinking is {{replacement approach}}, but we'd like a second opinion on whether
> that's the right pattern — and if you've seen other customers make the same move, what it
> actually cost them.
