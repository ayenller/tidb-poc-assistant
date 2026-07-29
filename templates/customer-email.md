# Customer Email Template

> Ready to send with light editing. Different from the report: the report is the reference
> document, this is the message. Keep it to roughly one screen — the report carries the
> detail and goes attached.
>
> Rules for filling this in:
> - **Do not hide the blockers.** Red flags and conflicts belong in the email, not only in
>   the attachment. Discovering a blocker three weeks into a PoC destroys more trust than
>   naming it on day one.
> - **Do not state prices or SLA numbers.**
> - **Label preview and beta features as such**, in the email itself.
> - **End with a small, specific ask.** "Let me know your thoughts" produces no reply; three
>   named questions produce three answers.

---

**Subject**: TiDB PoC — proposed approach for {{Customer}} {{project}}

Hi {{name}},

Thanks for walking me through your {{source database}} setup. Here's what I've put together
based on what you shared — the full assessment is attached, and this is the summary.

**What I understood**

- {{Source}}, roughly {{volume}}, peaking around {{QPS}}
- {{Continuous replication requirement, in their words}}
- {{The one or two feature requirements that mattered most}}
- {{Cloud and region}}

Please correct me if any of that is off — a few of these I inferred rather than had
confirmed, and {{the highest-impact assumption}} in particular would change the plan if it's
wrong.

**What I'd recommend**

**{{Offering}}** — {{one sentence on why, tied to their specific constraint rather than to a
feature list}}.

{{One short paragraph if a preview/beta feature is involved: name it, say it's in preview,
and say what that means practically for the PoC.}}

**On the data migration**

{{Two or three sentences. The path, the expected downtime at cutover, and the one thing you
need from them — usually the source binlog settings or the network path, since those involve
their team and set the schedule.}}

**Things we'll need to work through together**

{{Only include this section when there are red flags or a conflict. Be direct, and pair each
item with the path forward.}}

- **{{Feature}}** — TiDB doesn't support this. The usual approach is {{replacement}}. To size
  the work, could you tell me {{the estimation question}}?
- {{If there's a tier conflict: state it plainly, name the options, and say which one you'd
  lean toward and why.}}

**One thing to start now**

{{Include unless billing is already configured. Keep it short and non-pushy — it is a
logistics item, not a sales one. Name the route and who needs to do it.}}

To provision the cluster we'll need a payment method on your TiDB Cloud organization —
either a card added in the console, or a subscription through {{AWS / Azure / Google Cloud}}
Marketplace if you'd rather it go against your existing cloud spend. Worth starting early
since it usually needs {{whoever holds Organization Owner or Billing Manager}} rather than
the technical team, and it's the one item that can hold up kickoff for reasons that have
nothing to do with the technology.

{{If PoC credits are already running with no payment method — add: Also worth noting that if
the PoC credits lapse before a payment method is in place, clusters are recycled after 3 days
and backups after 7, so we'd want it sorted before {{date}}.}}

**What I need from you**

1. {{Specific question}}
2. {{Specific question}}
3. {{Specific question}}

Happy to jump on a call if that's faster than email.

Best,
{{name}}

---

## Variant — when there is a requirement conflict

Replace the "What I'd recommend" section with:

> **Where things stand**
>
> There's a tension in the requirements I want to flag before we design around it: {{plain
> statement of the conflict}}. {{Offering X}} covers {{requirement A}}, but {{requirement B}}
> rules it out.
>
> {{When the conflict involves vector search: Before we redesign anything, it's worth
> confirming this with our product team — the documentation is inconsistent on which tiers
> support it, and the constraint may not actually apply. I'm checking.}}
>
> If it does hold, there are three ways forward:
>
> 1. **{{Route A}}** — {{one line}}. Trade-off: {{cost/complexity}}.
> 2. **{{Route B}}** — {{one line}}. Trade-off: {{cost/complexity}}.
> 3. **{{Route C}}** — {{one line}}. Trade-off: {{cost/complexity}}.
>
> Given {{their stated priority}}, I'd lean toward {{route}} — but this is a call about your
> architecture, so I'd rather talk it through than assume.

---

## Variant — when the source is not MySQL-compatible

Add after the migration paragraph:

> Since you're coming from {{source}}, the schema and data conversion needs a closer look
> than I can give it from this conversation — type mapping and {{source-specific concern}}
> in particular. I'll run that assessment separately and come back with specifics rather
> than guess at the effort now.
