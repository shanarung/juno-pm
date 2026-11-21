# AI PRD · Juno

Nov 21, 2026 · @Shana Rungsarangnont

## Problem & user

Juno replaces manual, one-by-one triage of AI use cases with a consistent, value-based ranking that the AI product team can accept or override.

**The problem.** Today a coordinator reviews each use case as it arrives and manually assumes a priority. This produces inconsistent rankings and takes a lot of triage time.

**The user.** The AI product team uses Juno day to day. The product owner sets the final priority.

**What Juno does.** The prototype prioritizes use cases on the AI product management board (Asana, AIBLE Projects & Use Cases) from the stakeholder's inputs, mainly their value statements. It builds a dashboard that ranks the use cases. A human sets the priority in the dashboard, and that choice resets the priority on the Asana board.

**Prioritization strategy.** Ranking rests on three value pillars, which must persist:

1. Productivity: hours saved are mapped to a set hourly rate of $100/hr.
2. Net new capabilities.
3. Revenue and cost savings.

Juno also recommends a higher priority when its analysis shows a use case has higher value or scales further. Where the data has gaps, Juno does not skip the use case: it analyzes it and recommends from what it knows, except when the value statement is missing (see Gaps below).

**How the pillars rank.** Use cases rank in this order. First, by highest dollar savings: revenue and cost savings count as dollars, and productivity counts once hours are converted at $100/hr. Second, net new capabilities, which are flagged for human prioritization. Tickets that are launched or cancelled drop out and are not considered for prioritization.

**Gaps.** A ticket that is missing a value statement cannot be prioritized. Juno labels it as a gap in the data instead of ranking it, and the ticket stays blocked from prioritization until the value is captured.

**North star.** On each run, Juno goes through 100% of the in-scope tickets and, for each one, either prioritizes it or identifies the gaps needed to prioritize it.

**Blockers.** Use cases that expose PII, or that need legal review, are flagged. Juno reviews each use case and judges whether it could involve PII or have legal implications. A flagged use case is still ranked and recommended.

## The harness

Juno reads everything on one Asana board, ranks it daily, and changes the board only when a human confirms. Items marked *open* are not yet decided (details under Open questions).

| Surface | Specification |
| --- | --- |
| 01 Context | Everything on the AIBLE Projects & Use Cases board: line items, comments, custom fields, intake form answers, attachments, linked docs. Live on every load. |
| 02 Tools | Read the board, build the dashboard, write a human-confirmed priority back to Asana, post comments on tickets that record its recommendation. Tagging stakeholders is allowed (a tag is not communication). |
| 03 Loop | Scheduled run at 6 AM Pacific local time, Monday to Friday. Two attempts per full run; the second is a retry the user approves. Each attempt stops after 5 minutes. Stops and hands back to a human on an Asana connection failure or a timeout. |
| 04 Memory | Value pillars (team scope), human priority overrides, and the AI product team's prioritization patterns. Pillars and learned patterns never expire; overrides last until the ticket is launched or cancelled. |
| 05 Permissions | Priority write-back needs human confirmation. Read: everything on the board is readable. Draft and comment tiers: open. |
| 06 Verification | Before output ships: 100% of tickets processed, excluding launched or cancelled ones, and comments since the last board refresh picked up. Failures show as a banner and a per-ticket error state. |

## 01 Context · Data Requirements

Juno sees everything on the AIBLE Projects & Use Cases Asana board, fresh on every load.

- **Required sources:** the Asana board AIBLE Projects & Use Cases only. Everything on it is readable to Juno: line items, comments, custom fields, intake form answers, attachments, and linked docs. The $100/hr rate for productivity value is a set input.
- **Deliberate exclusions:** none. The older intake tickets (Sep 29 to Oct 22) and the AIBLE Projects & Use Cases x WRITER Team board are out of scope because only the one board is named.
- **SLA of truth:** live on every load. A change in Asana appears in the dashboard the next time it loads.
- **When a source is unreachable:** Juno shows the last known data set, clearly marked stale, and tells the user why the connection failed (for example an auth issue or Asana being down). It does not retry on its own; it asks the user whether to try again.

## 02 Tools · System Capabilities

Juno can read the board, rank and flag use cases, show a dashboard, write a confirmed priority back to Asana, and comment on tickets.

- **The verb list:**

| Verb | What it does | Side-effect class | Status |
| --- | --- | --- | --- |
| read\_board | Reads all line items, comments, custom fields, form answers, attachments, and linked docs | read | Stated |
| analyze\_and\_rank | Scores each use case against the three value pillars and scale, and recommends a priority | draft | Class needs confirming |
| flag\_risk | Flags PII exposure and legal-review needs on a use case, and flags net new capability use cases for human prioritization | draft | Class needs confirming |
| build\_dashboard | Shows the ranked use cases, flags, and any errors | draft | Class needs confirming |
| write\_priority | Writes the human-set priority (High, Medium, or Low) to the Asana Priority field | write | Stated: confirm |
| post\_comment | Writes a comment into an Asana ticket recording Juno's recommendation, so the team can see who changed the priority | write or send | Class and tier open |
| tag\_stakeholder | Tags a stakeholder in a comment | send | Stated: a tag is not communication. Class and tier open. |

- **Deliberate omissions:** Juno does not communicate with stakeholders, so it has no tool to ask them follow-up questions. It has no tool to close or delete tickets, or to edit fields that stakeholders submitted. It also does not recommend which platform (Writer, Claude, or Snowflake) a use case should be built on; that is out of scope for the product.

## 03 Loop · AI Costs & Latency

Juno runs on a schedule, gets two attempts per full run, and hands back to a human the moment it cannot reach Asana.

- **Trigger:** scheduled daily at 6 AM Pacific local time, Monday to Friday (PDT in summer, PST in winter). The dashboard's board data is live on every load, but a dashboard load does not re-run the analysis. A change made after the run, such as a new comment or an added value statement, is analyzed at the next scheduled run.
- **Turn ceiling:** two attempts per full run. If an Asana attempt fails, Juno does not retry on its own. It tells the user why it failed (for example an auth issue or Asana being down) and asks whether to try again. The second attempt is that user-approved retry. An attempt stops after 5 minutes, and a timeout uses one of the two attempts. Juno shows the tickets it finished, labels the rest "not processed", and asks whether to try again.
- **Escalation trigger:** an Asana connection failure or a timeout. Juno stops and hands back to a human.
- **Latency and cost target:** none for now. Thresholds will be set from the prototype's measured performance.

## 04 Memory · Data Requirements

Juno remembers the value pillars, human priority overrides, and how the team prioritizes, and a human override always outranks the model.

- **What persists, at what scope:**
  - The three value pillars. Only the AI product team can change them.
  - Human priority overrides.
  - The patterns in how the AI product team prioritizes. Juno analyzes these and keeps improving its suggestions for the final output. Juno learns from any change a human makes to a priority, and treats the final priority as the team's decision once the ticket moves through the workflow.
- **Expiry:** the value pillars and the learned prioritization patterns never expire. Human priority overrides last until the ticket reaches Done, meaning its status is launched or cancelled.
- **Write rules:** the human outranks the model. An override holds until the human changes it, even if the ticket's comments change and Juno's later analysis disagrees.

## 05 Permissions · AI Risks & Mitigations

The only tier decided so far is that writing a priority back to Asana needs a human confirmation.

| Side-effect class | Tier |
| --- | --- |
| read | Everything on the board is readable to Juno. Tier label (auto) to confirm. |
| draft | Open |
| write | confirm: the human's click in the dashboard confirms the priority write-back |
| send | Open: Juno may post comments and tag stakeholders, but must not communicate with stakeholders |

**Justification.** The priority write-back is tiered confirm because the human sets the final priority and that choice resets the priority on the Asana board. Blast radius and recoverability for the draft and send tiers cannot be written until those tiers are decided.

**Risk flags.** Use cases that expose PII are flagged, and use cases that need legal review are flagged. Juno decides by reviewing the use case itself and analyzing whether it could involve PII or have legal implications. A flagged use case is still ranked and recommended. Legal review is a visible flag only.

## 06 Verification · AI Testing & Measurement

Before the dashboard ships, Juno checks that it covered every ticket and picked up every new comment, and any failure is shown on screen. Every ticket shows a status: ranked, gap, or error.

- **The check before output ships:**
  1. Coverage: Juno ran through 100% of the use cases on the board, except those launched or cancelled, which are excluded from prioritization. No ticket is skipped. Every in-scope ticket is either prioritized or labelled with the gaps needed to prioritize it (the north star).
  2. Freshness: Juno took into account comments added or updated since the last time the board was refreshed, compared by timestamp.
- **Failure behaviour:** when a check fails, the dashboard says so explicitly in both places: a banner for the run as a whole, and an error state on each affected ticket, labelled with its error. Silently dropping a ticket or output is never the answer. Tickets not reached before a timeout are labelled "not processed". If the cause is an Asana connection failure, the dashboard shows the last known data marked stale, gives the reason, and asks the user whether to retry.

## Eval plan

Stub. Module 5 fills this in: the human eval rubric in `05-evals/human-rubric.md`, with its pass bar.

## Out of scope

These are decisions on the record, not omissions.

- Recommending which platform (Writer, Claude, or Snowflake) a use case should be built on.
- Communicating with stakeholders, including asking them follow-up questions.
- Closing or deleting tickets.
- Editing fields, outside of priority, that stakeholders submitted.
- Any capability not in the verb list.

No other exclusions are known yet, and no action is tiered `blocked` yet.


**Other**

- [ ] Latency and cost thresholds are open until the prototype is measured.
