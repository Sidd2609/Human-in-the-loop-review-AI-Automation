# Human in the Loop Review in AI Automation — PRD

**Owner:** Siddharth Agarwal
**Status:** Draft
**Version:** 0.1
**Last updated:** 2026-10-17

---

## 1. Summary

A reusable platform capability that decides which outputs of a multi-stage AI automation pipeline need human review, and which can proceed automatically. For each output it combines hard rules, policy checks, model-uncertainty signals, impact, entity risk (including bias) and random sampling to assign a review tier, and it presents flagged items to human reviewers with risk-prioritised queues and evidence rather than verdicts. It is use-case agnostic: each use case (the first being an airline content management system, with a retail marketplace later) configures its own signals, gates, thresholds and capacity. It exists because fully automated AI at scale will make errors, and humans cannot check every output without becoming a bottleneck or a passive approver.

## 2. Problem & outcome

### 2.1 Problem

Fully automated AI systems deployed at scale will make errors. Humans cannot check every AI output, so reviewing everything either stalls throughput (bottleneck) or degrades into approving without checking (rubber-stamping). Organisations therefore need a defensible way to decide which outputs a human must check, and on what parameters: confidence score, flagged language, tags or policy violations, random sampling, bias, and others that depend on the use case.

The pipeline this PRD targets is a content management system running AI at every stage: idea filtering, audience understanding, content creation (text, image, video), governance approval, and distribution to all or selected channels, with content personalised to individual customer behaviour.

### 2.2 Why now

TBD — see OQ-01.

### 2.3 Outcome

Human review effort is spent on the outputs that carry the most risk, review is demonstrably real rather than nominal, and the share of outputs that proceed automatically grows over time as evidence accumulates. Measured by the metrics in §3.

## 3. Success metrics

| Metric | Baseline | Target | Timeframe |
|---|---|---|---|
| Escaped-error rate (errors found in random samples of auto-approved outputs) | TBD — needs baseline measurement | TBD | TBD |
| Review precision (share of reviewed items edited or rejected) | TBD — needs baseline measurement | TBD | TBD |
| Reviewer catch rate on seeded errors | TBD — needs baseline measurement | TBD | TBD |
| Review queue age and SLA attainment | TBD — needs baseline measurement | TBD | TBD |
| Auto-approve rate by tier and category | TBD — needs baseline measurement | TBD | TBD |
| Time-to-publish | TBD — needs baseline measurement | TBD | TBD |
| Downstream incident rate (complaints, unsubscribes, legal flags) | TBD — needs baseline measurement | TBD | TBD |

Baselines and targets are tracked in OQ-02.

## 4. Personas

- **Campaign Manager**: creates and launches AI-assisted campaigns; wants speed without incidents.
- **Reviewer**: a generalist who reviews held items in the review queue.
- **Specialist Reviewer**: legal, brand, compliance or loyalty expert who reviews Tier 3 items.
- **Risk & Policy Owner**: owns the policy knowledge base, tier thresholds and acceptable error rates; approves configuration activation.
- **Platform Administrator**: configures a use case and operates suppression.

## 5. Core principle

Route by risk, not by default. Hard gates override scores. Reviewers are shown evidence, not verdicts.

Requirements below that refer to "the core principle" are constrained by this rule: no confidence or score value may bypass a hard gate, and no review screen may display an AI-generated pass/fail verdict.

## 6. Scope boundary

### 6.1 In scope

- Signal capture for outputs at each pipeline stage
- Hard-gate rules and policy compliance checks
- Risk scoring and tier assignment
- Random sampling and seeded errors
- Review queues, risk priority tagging and the reviewer workbench
- Decision logging, metrics and recalibration inputs
- Use case configuration and shadow mode
- Suppression and kill switch
- Audit trail

### 6.2 Out of scope

**Deferred to a later phase**
- Automated learning of signal weights from labelled decisions (this version records the labels and supports manual recalibration)
- A second configured use case (retail marketplace)

**Not planned**
- Building the AI generators for ideas, audiences or content
- Distribution channel infrastructure (sending email, push, SMS or media)

## 7. User journeys

### 7.1 Output flows through auto-approve

1. A pipeline stage (idea, audience, content, governance or distribution) produces an output, and the platform captures its signal bundle.
2. The platform evaluates the hard gates. None fire.
3. The platform computes the risk score and assigns tier 0 or tier 1.
4. The output is auto-approved and proceeds to the next stage.
5. For tier 1, and at a small rate for tier 0, the platform pulls a random sample into the review queue for QA.
6. The platform logs the full signal breakdown and the routing decision.

### 7.2 Output is flagged and reviewed by a human

1. A hard gate fires or the score places the output in tier 2 or tier 3, and the output is held.
2. The platform places it in the Reviewer queue (tier 2) or the Specialist Reviewer queue (tier 3), tagged High, Medium or Low risk.
3. The reviewer opens the item and sees the top contributing signals and evidence (matched policy clause, diff, highlighted risky span) but no pass/fail verdict.
4. The reviewer chooses approve, edit or reject, with a reason category.
5. The platform releases, revises or blocks the output, and logs the decision, reviewer and time spent.
6. The decision is added to the metrics and recalibration data.

### 7.3 Platform Administrator configures a use case

1. The administrator creates a use case (for example the airline CMS) and selects its pipeline stages.
2. They define hard gates, off-limits attributes and the regulated lexicon.
3. They set signal weights, impact scales and tier thresholds.
4. They set sampling rates, the seeded-error rate and review capacity.
5. They run the configuration in shadow mode, where humans review every output and the platform records the decisions.
6. They review the calibration results and activate the configuration with the Risk & Policy Owner's approval.

### 7.4 Kill switch during an incident

1. An external event occurs, such as an operational disruption or crisis.
2. An authorised user triggers suppression for a scope (route, region, segment or all).
3. The platform immediately holds all matching pending and scheduled outputs.
4. The platform notifies the Campaign Manager and the Risk & Policy Owner.
5. After the incident, an authorised user lifts the suppression and held outputs are re-routed through scoring.
6. The platform records the event in the audit trail.

## 8. Requirements

> **This is the contract.** Once this PRD is approved, these acceptance criteria are the authoritative definition of "done". Amendments after approval follow the major-version and re-approval rule.

### 8.1 Signal capture & gates

#### REQ-01 — Capture signal bundle per output

**Serves:** §7.1 step 1; §7.2 step 1

**User story**

*As a Risk & Policy Owner, I want every output to carry a recorded set of risk signals, so that routing decisions are based on evidence I can inspect.*

**Acceptance criteria**

1. **GIVEN** a pipeline stage configured for a use case **WHEN** it produces an output **THEN** the platform creates a signal bundle for that output before any routing decision is made.
2. **GIVEN** a use case that enables the signal families deterministic rules, policy compliance, model uncertainty, novelty, impact, reversibility, entity risk and behavioural anomaly **WHEN** a signal bundle is created **THEN** it contains a value or an explicit "unavailable" marker for each enabled signal.
3. **GIVEN** a signal bundle containing an "unavailable" marker for a signal the use case marks as required **WHEN** routing is evaluated **THEN** the output is held for human review and is not auto-approved.

#### REQ-02 — Evaluate hard gates

**Serves:** §7.1 step 2; §7.2 step 1

**User story**

*As a Risk & Policy Owner, I want non-negotiable rules to force review or blocking regardless of any score, so that high-confidence outputs cannot bypass them (core principle, §5).*

**Acceptance criteria**

1. **GIVEN** an output with a signal bundle **WHEN** routing is evaluated **THEN** all configured hard gates are evaluated before the risk score is used to assign a tier.
2. **GIVEN** a hard gate whose action is block **WHEN** it fires **THEN** the output is blocked regardless of its risk score.
3. **GIVEN** a hard gate whose action is mandatory review **WHEN** it fires **THEN** the output is placed in the queue named by the gate and is not auto-approved regardless of its risk score.
4. **GIVEN** an output matching a term in the regulated lexicon **WHEN** routing is evaluated **THEN** the hard gate for that term fires and the matched term is recorded in the signal bundle.
5. **GIVEN** more than one hard gate fires on the same output **WHEN** routing is evaluated **THEN** the most restrictive action applies (block, then Specialist Reviewer review, then Reviewer review) and every fired gate is recorded.

#### REQ-03 — Check outputs against the policy knowledge base

**Serves:** §7.1 step 2; §7.2 step 3

**User story**

*As a Risk & Policy Owner, I want each output checked against the relevant policies in the policy knowledge base, so that policy changes take effect without rewriting rules.*

**Acceptance criteria**

1. **GIVEN** an output **WHEN** the policy check runs **THEN** the platform retrieves the policy clauses relevant to the output and records each retrieved clause identifier in the signal bundle.
2. **GIVEN** a policy check result of definite violation **WHEN** routing is evaluated **THEN** the output is blocked.
3. **GIVEN** a policy check result of possible violation **WHEN** routing is evaluated **THEN** the output is routed to tier 2 or higher.
4. **GIVEN** a policy check that retrieves no clauses **WHEN** routing is evaluated **THEN** the signal bundle records "no policy coverage" and the output is treated according to the use case's configured no-coverage action.
5. **IF** the policy knowledge base is updated **THEN** the next output evaluated uses the updated version and the signal bundle records the knowledge base version.

#### REQ-04 — Check entity risk and bias

**Serves:** §7.1 step 2; §7.2 step 1

**User story**

*As a Risk & Policy Owner, I want targeting and outcomes checked for off-limits attributes and bias, so that personalisation never uses protected or sensitive inferences and does not disadvantage groups.*

**Acceptance criteria**

1. **GIVEN** an output that is an audience definition or a personalised piece of content **WHEN** it uses an attribute on the use case's off-limits list **THEN** the output is blocked and the attribute is recorded.
2. **GIVEN** an output targeting a segment that includes minors or a configured vulnerable group **WHEN** routing is evaluated **THEN** the entity risk signal is set to its maximum value.
3. **GIVEN** a use case with a bias check configured for an audience definition **WHEN** the selected audience differs from the eligible population on a configured group dimension by more than the configured threshold **THEN** the platform flags a bias signal and routes the output to tier 2 or higher.
4. **GIVEN** a bias flag **WHEN** a reviewer opens the item **THEN** the group dimension and measured difference are shown as evidence.

### 8.2 Risk scoring & tiering

#### REQ-05 — Compute risk score

**Serves:** §7.1 step 3

**User story**

*As a Platform Administrator, I want a risk score that reflects both how likely an output is to be wrong and how costly an error would be, so that the same uncertainty produces stricter routing for high-impact outputs.*

**Acceptance criteria**

1. **GIVEN** an output whose hard gates do not block **WHEN** routing is evaluated **THEN** the platform computes a risk score from a likelihood-of-error component and an impact component.
2. **GIVEN** two outputs with identical likelihood-of-error signals **WHEN** one has a higher impact value **THEN** that output receives a risk score greater than or equal to the other.
3. **GIVEN** two outputs with identical impact **WHEN** one has a higher value on any likelihood-of-error signal **THEN** that output receives a risk score greater than or equal to the other.
4. **IF** the impact component would otherwise fall below the configured minimum impact **THEN** the configured minimum impact is used.
5. **GIVEN** a computed risk score **WHEN** it is stored **THEN** each contributing signal value, each impact dimension value and the final score are stored with the output.

#### REQ-06 — Assign tier and route

**Serves:** §7.1 steps 3–4; §7.2 steps 1–2

**User story**

*As a Campaign Manager, I want each output routed automatically to the right level of review, so that low-risk outputs proceed without waiting and high-risk ones are held.*

**Acceptance criteria**

1. **GIVEN** a risk score and the use case's tier thresholds **WHEN** routing is evaluated **THEN** the output is assigned exactly one tier: 0 (auto-approve), 1 (auto-approve with sampling), 2 (Reviewer queue), 3 (Specialist Reviewer queue) or 4 (block).
2. **GIVEN** an output assigned tier 0 or tier 1 **WHEN** no hard gate has fired **THEN** the output proceeds to the next pipeline stage without human action.
3. **GIVEN** an output assigned tier 2 or tier 3 **WHEN** routing completes **THEN** the output is held and does not proceed to the next pipeline stage until a reviewer decision is recorded.
4. **GIVEN** an output assigned tier 4 **WHEN** routing completes **THEN** the output is blocked and the Campaign Manager is notified of the block and its reasons.
5. **GIVEN** the same risk score for two outputs with different impact classes **WHEN** the use case defines impact-scaled thresholds **THEN** each output is assigned the tier defined for its impact class.
6. **GIVEN** a tier assignment **WHEN** it differs from the tier the risk score alone would give because a hard gate applied **THEN** both tiers and the gate are recorded.

#### REQ-07 — Default new things to human review

**Serves:** §7.1 step 3

**User story**

*As a Risk & Policy Owner, I want anything new to the system reviewed by a human at first, so that no new template, segment or model is trusted before it has evidence behind it.*

**Acceptance criteria**

1. **GIVEN** a use case configured with a new-item review count N **WHEN** an output uses a template, segment or generator version that has fewer than N prior reviewed outputs **THEN** the output is assigned at least tier 2 regardless of its risk score.
2. **GIVEN** a template, segment or generator version that has reached N reviewed outputs **WHEN** routing is evaluated **THEN** the new-item rule no longer applies to it.

### 8.3 Sampling & seeded errors

#### REQ-08 — Random sampling of auto-approved outputs

**Serves:** §7.1 step 5

**User story**

*As a Risk & Policy Owner, I want a random share of auto-approved outputs reviewed, so that I have an unbiased measure of the real error rate.*

**Acceptance criteria**

1. **GIVEN** an output assigned tier 1 **WHEN** routing completes **THEN** it is selected for QA review with the configured tier 1 sampling probability.
2. **GIVEN** an output assigned tier 0 **WHEN** routing completes **THEN** it is selected for QA review with the configured tier 0 sampling probability.
3. **GIVEN** a configured sampling stratification by category and template **WHEN** samples are selected **THEN** each stratum receives at least the configured minimum number of samples per period.
4. **GIVEN** an output selected for QA sampling **WHEN** it is selected **THEN** it has already proceeded to the next pipeline stage, and the QA outcome does not delay it.
5. **GIVEN** a QA review that finds an error **WHEN** the decision is recorded **THEN** the output is counted as an escaped error and the owning stakeholders for that use case are notified.
6. **GIVEN** a category whose escaped-error rate exceeds its configured limit, or whose generator or prompt version has changed **WHEN** the next period begins **THEN** the sampling probability for that category is increased to its configured elevated rate.

#### REQ-09 — Seed known errors into review queues

**Serves:** §7.2 steps 3–4

**User story**

*As a Risk & Policy Owner, I want known-bad items mixed into review queues, so that I can measure whether review is real.*

**Acceptance criteria**

1. **GIVEN** a configured seeded-error rate **WHEN** items enter tier 2 and tier 3 queues **THEN** known-bad items are inserted at that rate.
2. **GIVEN** a seeded item **WHEN** a reviewer opens it **THEN** it is indistinguishable from a real item in the reviewer interface.
3. **GIVEN** a seeded item **WHEN** the reviewer decides **THEN** the platform records whether the reviewer correctly identified the seeded error, and the seeded item is never released to a channel.
4. **GIVEN** a reviewer's seeded-item results over a period **WHEN** the catch rate falls below the configured minimum **THEN** the Risk & Policy Owner is notified.

### 8.4 Reviewer queue & workbench

#### REQ-10 — Tag and order review queue items by risk priority

**Serves:** §7.2 step 2

**User story**

*As a Reviewer, I want every queue item tagged High, Medium or Low risk, so that I can prioritise the items that matter most.*

**Acceptance criteria**

1. **GIVEN** an output held in a tier 2 or tier 3 queue **WHEN** it is placed in the queue **THEN** it carries exactly one risk priority tag: High, Medium or Low.
2. **GIVEN** the use case's configured risk-priority band boundaries **WHEN** the tag is assigned **THEN** it is derived from the item's risk score and the same score always yields the same tag.
3. **GIVEN** a queue **WHEN** a reviewer opens it **THEN** items are ordered High before Medium before Low, and within a tag by oldest first.
4. **GIVEN** a queue **WHEN** a reviewer applies a risk priority tag filter **THEN** only items with the selected tags are shown.
5. **GIVEN** a queue item **WHEN** it is displayed in the list **THEN** the list shows its risk priority tag, tier, pipeline stage, the top flagged reason and its time in queue.
6. **GIVEN** a Low-tagged item whose time in queue exceeds the configured maximum **WHEN** the threshold is crossed **THEN** its display order is raised above newer Low items and the breach is recorded.

#### REQ-11 — Show evidence, not a verdict

**Serves:** §7.2 step 3

**User story**

*As a Reviewer, I want to see why an item was flagged and the supporting evidence, so that I check something specific without being told the answer (core principle, §5).*

**Acceptance criteria**

1. **GIVEN** a reviewer opening a held item **WHEN** the item loads **THEN** it displays the output, the top contributing signals by contribution and the evidence for each (matched lexicon term, retrieved policy clause, verification mismatch, bias measure, or highlighted span).
2. **GIVEN** a held item **WHEN** it is displayed **THEN** no AI-generated approve or reject recommendation is shown.
3. **GIVEN** an output that is a revision or an edit of earlier content **WHEN** it is displayed **THEN** the change from the earlier version is shown as a diff.
4. **GIVEN** a held item **WHEN** it is displayed **THEN** the numeric risk score is not displayed as a verdict, and the risk priority tag is shown.

#### REQ-12 — Record a structured reviewer decision

**Serves:** §7.2 step 4

**User story**

*As a Reviewer, I want to approve, edit or reject with a reason, so that my decision is deliberate and useful for improving the system.*

**Acceptance criteria**

1. **GIVEN** a held item **WHEN** the reviewer submits a decision **THEN** the decision is exactly one of approve, edit or reject.
2. **GIVEN** a reject decision or an edit decision **WHEN** the reviewer submits **THEN** a reason category from the use case's configured list is required and the submission is refused without one.
3. **GIVEN** an edit decision **WHEN** the reviewer submits **THEN** the edited output is stored and the difference from the original is recorded.
4. **GIVEN** a tier 3 item **WHEN** a user without the Specialist Reviewer role attempts to decide **THEN** the decision is refused.
5. **GIVEN** a reviewer opening an item **WHEN** they submit a decision **THEN** the time between opening and submission is recorded.

#### REQ-13 — Act on the reviewer decision

**Serves:** §7.2 step 5

**User story**

*As a Campaign Manager, I want a reviewer decision to take effect automatically, so that approved work moves on and rejected work does not.*

**Acceptance criteria**

1. **GIVEN** an approve decision **WHEN** it is recorded **THEN** the output is released to the next pipeline stage.
2. **GIVEN** an edit decision **WHEN** it is recorded **THEN** the edited output is released to the next pipeline stage in place of the original.
3. **GIVEN** a reject decision **WHEN** it is recorded **THEN** the output is blocked and the Campaign Manager is notified with the reason category.
4. **GIVEN** an edit decision that changes content which a hard gate applies to **WHEN** it is recorded **THEN** the edited output is re-evaluated by the hard gates before release.

#### REQ-14 — Detect passive approval

**Serves:** §7.2 steps 3–5

**User story**

*As a Risk & Policy Owner, I want signs of rubber-stamping surfaced, so that I can intervene before oversight becomes nominal.*

**Acceptance criteria**

1. **GIVEN** a reviewer decision **WHEN** the time spent is below the configured minimum for that item's content length **THEN** the decision is flagged as a fast approval in the reviewer's activity record.
2. **GIVEN** a reviewer with a run of consecutive approvals at or above the configured streak length **WHEN** the streak is reached **THEN** the Risk & Policy Owner is notified.
3. **GIVEN** a configured maximum session length or queue batch size **WHEN** a reviewer reaches it **THEN** the platform prompts a break before showing further items.
4. **GIVEN** a queue **WHEN** the share of flagged items approved unchanged over a period exceeds the configured limit **THEN** the filter-looseness metric is flagged for the Risk & Policy Owner.

#### REQ-15 — Alert on queue capacity breach

**Serves:** §7.2 step 2

**User story**

*As a Platform Administrator, I want to know when the review queue exceeds capacity, so that I can adjust thresholds deliberately instead of letting a backlog build.*

**Acceptance criteria**

1. **GIVEN** a configured review capacity per queue **WHEN** the number of held items or the oldest item age exceeds it **THEN** the Platform Administrator and Risk & Policy Owner are notified.
2. **GIVEN** a capacity breach **WHEN** the notification is sent **THEN** it includes the queue, the count, the oldest item age and the breakdown by risk priority tag.

### 8.5 Feedback, calibration & metrics

#### REQ-16 — Capture decisions as calibration data

**Serves:** §7.2 step 6

**User story**

*As a Risk & Policy Owner, I want every human decision linked to the signals that produced the routing, so that thresholds and weights can be recalibrated from evidence.*

**Acceptance criteria**

1. **GIVEN** a recorded reviewer or QA decision **WHEN** it is saved **THEN** it is linked to the output's full signal bundle, tier, risk priority tag and the reviewer's decision, reason category and edit.
2. **GIVEN** calibration data for a use case **WHEN** an authorised user requests an export **THEN** the platform provides the linked records for the requested period in a downloadable file.

#### REQ-17 — Report metrics

**Serves:** cross-cutting

**User story**

*As a Risk & Policy Owner, I want the system's key quality and load measures reported, so that I can judge whether review is working and whether autonomy can grow.*

**Acceptance criteria**

1. **GIVEN** a use case with decisions recorded **WHEN** a user views metrics **THEN** the platform reports escaped-error rate, review precision, seeded-error catch rate, queue age and SLA attainment, auto-approve rate by tier and category, and time-to-publish, each for a selectable period.
2. **GIVEN** a metric **WHEN** it is displayed **THEN** it can be filtered by pipeline stage, category, template and reviewer.
3. **GIVEN** a metric whose value crosses a configured alert threshold **WHEN** the period closes **THEN** the Risk & Policy Owner is notified.

#### REQ-18 — Propose tier changes from override evidence

**Serves:** cross-cutting

**User story**

*As a Risk & Policy Owner, I want the platform to propose promoting or demoting content categories based on review outcomes, so that autonomy expands only on evidence.*

**Acceptance criteria**

1. **GIVEN** a category with at least the configured minimum number of reviewed outputs **WHEN** its override rate is below the configured promotion limit **THEN** the platform proposes a lower review tier for that category.
2. **GIVEN** a category **WHEN** its override rate exceeds the configured demotion limit **THEN** the platform proposes a stricter review tier for that category.
3. **GIVEN** a proposal **WHEN** it is created **THEN** it takes effect only after the Risk & Policy Owner approves it.

### 8.6 Configuration & shadow mode

#### REQ-19 — Configure a use case

**Serves:** §7.3 steps 1–4

**User story**

*As a Platform Administrator, I want to configure signals, gates, thresholds and capacity for a use case, so that the same platform serves different businesses.*

**Acceptance criteria**

1. **GIVEN** a Platform Administrator **WHEN** they create a use case **THEN** they can select which pipeline stages it applies to.
2. **GIVEN** a use case **WHEN** configuring it **THEN** the administrator can define hard gates, the off-limits attribute list, the regulated lexicon, enabled signals with weights, impact scales, tier thresholds, risk-priority band boundaries, sampling rates, seeded-error rate, new-item review count and review capacity.
3. **GIVEN** a configuration with a missing required value or tier thresholds that are not in ascending order **WHEN** the administrator saves **THEN** the save is refused and the invalid fields are identified.
4. **GIVEN** a user without the Platform Administrator role **WHEN** they attempt to change a configuration **THEN** the change is refused.

#### REQ-20 — Run a configuration in shadow mode

**Serves:** §7.3 step 5

**User story**

*As a Risk & Policy Owner, I want a new configuration observed against full human review first, so that thresholds are set from evidence rather than guesses.*

**Acceptance criteria**

1. **GIVEN** a configuration in shadow mode **WHEN** an output is produced **THEN** the platform computes its signals, score and tier but routes the output to human review regardless of tier.
2. **GIVEN** a configuration in shadow mode **WHEN** a reviewer decides **THEN** the platform records both the reviewer decision and the tier the configuration would have assigned.
3. **GIVEN** a shadow period **WHEN** an authorised user requests the calibration report **THEN** it shows, per tier, the count of outputs and the share the reviewers edited or rejected.

#### REQ-21 — Version and activate a configuration

**Serves:** §7.3 step 6

**User story**

*As a Risk & Policy Owner, I want to approve a configuration before it goes live, so that no one changes routing rules alone.*

**Acceptance criteria**

1. **GIVEN** a saved configuration **WHEN** it is saved **THEN** it is stored as a new immutable version with author and time.
2. **GIVEN** a configuration version **WHEN** the Platform Administrator requests activation **THEN** it becomes active only after the Risk & Policy Owner approves it.
3. **GIVEN** an activation **WHEN** it takes effect **THEN** outputs routed after that moment record the new configuration version and outputs routed before it keep the earlier version.
4. **GIVEN** a prior configuration version **WHEN** an authorised user requests rollback **THEN** that version is re-activated, subject to the same approval.

### 8.7 Suppression & kill switch

#### REQ-22 — Trigger suppression

**Serves:** §7.4 steps 1–2

**User story**

*As a Platform Administrator, I want to suppress content for a scope when an external event occurs, so that inappropriate content does not go out during an incident.*

**Acceptance criteria**

1. **GIVEN** a user with the suppression-trigger role **WHEN** they trigger suppression **THEN** they must specify a scope (route, region, segment or all) and a reason.
2. **GIVEN** a user without the suppression-trigger role **WHEN** they attempt to trigger suppression **THEN** the attempt is refused.
3. **GIVEN** a configured integration with an external incident feed **WHEN** the feed reports an incident matching a configured suppression rule **THEN** suppression for the rule's scope is triggered and recorded as system-triggered.

#### REQ-23 — Hold matching outputs

**Serves:** §7.4 step 3

**User story**

*As a Campaign Manager, I want all affected content held at once, so that nothing inappropriate is sent while an incident is live.*

**Acceptance criteria**

1. **GIVEN** an active suppression **WHEN** it is triggered **THEN** all pending and scheduled outputs matching its scope are held and none are released to a channel while it is active.
2. **GIVEN** an active suppression **WHEN** a new output matching its scope reaches the distribution stage **THEN** it is held.
3. **GIVEN** outputs held by suppression **WHEN** an authorised user views them **THEN** they are listed with the suppression that holds them.

#### REQ-24 — Notify on suppression

**Serves:** §7.4 step 4

**User story**

*As a Risk & Policy Owner, I want the right people told immediately when suppression starts, so that the response is coordinated.*

**Acceptance criteria**

1. **GIVEN** a suppression **WHEN** it is triggered **THEN** the Campaign Managers with affected outputs and the Risk & Policy Owner are notified with the scope, reason and number of outputs held.

#### REQ-25 — Lift suppression and re-route

**Serves:** §7.4 step 5

**User story**

*As a Platform Administrator, I want held content re-assessed when an incident ends, so that stale content is not sent without a fresh check.*

**Acceptance criteria**

1. **GIVEN** an active suppression **WHEN** an authorised user lifts it **THEN** every output it held is re-evaluated through hard gates and scoring.
2. **GIVEN** a held output re-evaluated after lift **WHEN** its scheduled send time has passed **THEN** it is placed in the Reviewer queue and not sent automatically.
3. **GIVEN** a lift **WHEN** it is requested **THEN** the user must record a reason.

### 8.8 Audit & logging

#### REQ-26 — Keep an audit trail

**Serves:** §7.1 step 6; §7.2 step 5; §7.4 step 6

**User story**

*As a Risk & Policy Owner, I want a complete record of every routing decision, human decision, configuration change and suppression, so that any outcome can be explained later.*

**Acceptance criteria**

1. **GIVEN** any routing decision **WHEN** it is made **THEN** the platform records the output, signal bundle, hard gates fired, risk score components, tier, risk priority tag, configuration version and time.
2. **GIVEN** any human decision **WHEN** it is made **THEN** the platform records the user, decision, reason category, time spent and time of decision.
3. **GIVEN** any configuration change, activation, suppression trigger or lift **WHEN** it occurs **THEN** the platform records the user, scope, reason, before and after values and time.
4. **GIVEN** an audit record **WHEN** any user attempts to edit or delete it **THEN** the attempt is refused and recorded.
5. **GIVEN** a past output **WHEN** an authorised user requests a decision explanation **THEN** the platform shows the recorded signals and the steps that produced its tier.

## 9. Non-functional requirements

### NFR-01 — Role-based access

**Requirement:** Access to review queues, configuration, suppression, metrics and audit records is restricted by role (Reviewer, Specialist Reviewer, Risk & Policy Owner, Platform Administrator, Campaign Manager).
**Target / threshold:** Role-to-permission mapping TBD — see OQ-07.

### NFR-02 — Audit retention

**Requirement:** Audit records are retained and retrievable for a defined period.
**Target / threshold:** TBD — see OQ-06.

### NFR-03 — Routing decision latency and availability

**Requirement:** Routing decisions complete quickly enough not to block the pipeline, and the routing capability is available when pipeline stages produce outputs.
**Target / threshold:** TBD — see OQ-05.

### NFR-04 — Explainability

**Requirement:** Every routing decision can be explained from stored signals without re-running models.
**Target / threshold:** Satisfied by REQ-26/AC-5.

### NFR-05 — Failure behaviour

**Requirement:** When the routing capability cannot produce a decision, the output is held and is never auto-approved.
**Target / threshold:** Applies to all pipeline stages. Time to detect failure TBD — see OQ-05.

## 10. Open questions

### 10.1 Needs a decision (open)

| ID | Topic | Detail | Owner |
|---|---|---|---|
| OQ-01 | Why now | The trigger for this work and the cost of not acting is not yet stated. | Siddharth Agarwal |
| OQ-02 | Success metric baselines and targets | No baselines or targets exist for the §3 metrics. | Siddharth Agarwal |
| OQ-03 | Risk priority band boundaries | Score boundaries for High, Medium and Low tags (REQ-10) are undefined. | Risk & Policy Owner |
| OQ-04 | Acceptable error rate per tier | Needed to set tier thresholds and escaped-error limits. | Risk & Policy Owner |
| OQ-05 | Review capacity, SLA and platform latency | Reviewer headcount, review SLA, routing latency and availability targets are undefined (REQ-15, NFR-03, NFR-05). | Siddharth Agarwal |
| OQ-06 | Audit retention period | Required by regulation or policy for the airline use case. | Risk & Policy Owner |
| OQ-07 | Role-to-permission mapping and suppression-trigger authority | Who may trigger suppression and lift it. | Risk & Policy Owner |
| OQ-08 | Off-limits attributes for the airline use case | The concrete list (for example health, bereavement, minors). | Risk & Policy Owner |

### 10.2 Deferred to another document's owner

| ID | Topic | Owning document |
|---|---|---|
| OQ-09 | Scoring method, signal normalisation, calibration approach and default weights | `/technical-requirement` for this platform (starting point: the scoring formula discussed during discovery) |
| OQ-10 | Integration approach with the AI generators, rules engine, policy knowledge base and distribution channels | `/architecture` overview |

### 10.3 Resolved *(kept for traceability)*

| ID | Topic | Resolution |
|---|---|---|
| — | Bias as a separate requirement area | Resolved 2026-10-17 by Siddharth Agarwal: bias sits under entity risk (REQ-04). |

## 11. Dependencies

- **AI generators for ideas, audiences and content** — produce the outputs this platform routes.
- **Policy knowledge base** — supplies policies for the compliance check; owned by the Risk & Policy Owner.
- **Rules engine / reference data** — supplies verification of prices, entitlements and eligibility for the use case.
- **Distribution channels** — receive released outputs and must honour holds and suppression.
- **Identity provider** — supplies user identity and roles.
- **External incident feed** *(optional)* — supplies events for system-triggered suppression (REQ-22/AC-3).

## 12. Glossary

| Term | Meaning in this document |
|---|---|
| Output | Any item produced by a pipeline stage: an idea, audience definition, piece of content, governance result or distribution plan. |
| Pipeline stage | One of idea filtering, audience understanding, content creation, governance and approval, or distribution. |
| Use case | A configured deployment of the platform for one business (for example the airline CMS). |
| Signal | A measurable indicator of risk about an output. |
| Signal bundle | The full set of signal values recorded for one output. |
| Hard gate | A rule that forces blocking or mandatory review regardless of score. |
| Risk score | A number combining likelihood of error and impact for an output. |
| Impact | The estimated cost of an error, from reach, money or contractual exposure, reversibility, entity risk and visibility. |
| Tier | The routing class assigned to an output: 0 auto-approve, 1 auto-approve with sampling, 2 Reviewer queue, 3 Specialist Reviewer queue, 4 block. |
| Risk priority tag | High, Medium or Low label on a queued item, derived from its risk score. |
| Held | An output waiting for a human decision or released suppression; it does not proceed. |
| Auto-approve | Proceeding to the next stage without a human decision. |
| Escaped error | An error found in an auto-approved output by QA sampling or downstream. |
| Seeded error | A known-bad item inserted into a queue to test reviewer attention. |
| Override | A reviewer decision of edit or reject. |
| Shadow mode | Running a configuration while humans review every output, to collect calibration data. |
| Suppression | A scoped hold on all matching outputs during an incident. |
| Off-limits attribute | An attribute that must never be used for targeting. |
| Policy knowledge base | The versioned store of policies the compliance check retrieves from. |
| Review capacity | The number of items a queue can handle within its SLA. |

## 13. Risks & assumptions

- **Risk (product):** the checker and the generator share blind spots, so flagged-item evidence misses the same errors. **Mitigation:** independent checks in REQ-02 and REQ-03, plus random sampling (REQ-08).
- **Risk (product):** reviewers defer to the tier or tag as a verdict. **Mitigation:** evidence, not verdict, display (REQ-11); seeded errors (REQ-09); passive-approval detection (REQ-14).
- **Risk (delivery):** without shadow-period data, thresholds are guesses. **Mitigation:** REQ-20, with conservative new-item defaults (REQ-07).
- **Risk (regulatory):** a stale policy knowledge base lets violations pass. **Mitigation:** version recording (REQ-03/AC-5); ownership by the Risk & Policy Owner.
- **Assumption:** pipeline stages can expose their outputs and signals to this platform and honour holds.
- **Assumption:** an owner of the policy knowledge base exists for each use case.

## 14. Stakeholders & sign-off

| Role | Name | Approves? |
|---|---|---|
| Product Owner | Siddharth Agarwal | ☐ |

## 15. Revision history

| Date | Author | Change |
|---|---|---|
| 2026-10-17 | Siddharth Agarwal | Initial draft (v0.1) |
