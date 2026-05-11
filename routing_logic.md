# Routing logic — pre-registration document

The AI Co-Scientist Typology routes each survey respondent to one of nine
characters based on their answers. This document fully specifies the routing
so the procedure is reproducible from the survey instrument alone, and so any
weight choice can be questioned and revised.

> **Status**: this document encodes the routing logic intended for use on
> survey responses collected via the public Qualtrics survey at
> `cumc.co1.qualtrics.com/jfe/form/SV_9uWW9GgwPuRucoS`. The same logic is
> implemented in `route_responses.py` (Python, for analysis) and
> `qualtrics_routing.js` (JavaScript, for live respondent routing).

## 1. Theoretical foundation

The typology has two axes, both grounded in the human–computer-interaction and
human-factors literatures on agent-style automation:

- **Initiative** — *who proposes the next move.* Allen 1999, Horvitz 1999 on
  mixed-initiative interaction. Three values: user-initiated, mixed,
  system-initiated.
- **Scope** — *how much of a task the AI handles in one invocation.* Sheridan
  & Verplank 1978 on levels of automation; Parasuraman, Sheridan & Wickens
  2000. Three values: narrow function, broad function, whole task.

The 3 × 3 grid produces nine cells. Each cell is assigned a Tarot Major
Arcana archetype and a scientific instrument from the natural-philosophy
tradition; the choice of archetype/instrument is descriptive (it gives each
cell a vivid identity) and does not enter the routing computation.

| | User-initiated | Mixed-initiated | System-initiated |
|---|---|---|---|
| **Narrow function** | I — Magician (Lapidary) | II — Temperance (Adjuster) | III — Justice (Inspector) |
| **Broad function** | IV — Empress (Drafter) | V — Lovers (Collaborator) | VI — Wheel (Conductor) |
| **Whole task** | VII — Fool (Deputy) | VIII — Priestess (Co-Investigator) | IX — Hermit (Successor) |

## 2. Survey instrument

Routing draws from six questions in the survey. The remaining questions
(Q1a, Q1b, Q5, Q6 in any block, Q7, Q8, Q9, Q12) are *not* used in routing —
they serve as cross-check signals or interview-recruitment instruments.

| Question | Type | Role in routing |
|---|---|---|
| Q1c | single-select | scope + initiative *modulator* |
| Q2 | single-select | scope + initiative *modulator* |
| Q3 | multi-select | scope axis (averaged) + count modifier |
| Q4 | multi-select | initiative axis (summed) |
| Q10 | multi-select | scope axis (averaged) + initiative axis (summed) |
| Q11 | multi-select, top 3 | initiative axis (summed) |

## 3. Modulator weights

Modulators add a small offset to whichever axis they affect. They are bounded
in absolute value by ±0.3 so that no single modulator can override the
weighted-contributor signals.

### Q1c — DS/ML competence

| Selection | Scope offset | Initiative offset |
|---|---|---|
| Beginner: I am learning DS/ML methods | +0.3 | +0.2 |
| Intermediate: I can run analyses/models with some support | 0 | 0 |
| Advanced: I independently design and execute DS/ML projects | −0.1 | −0.1 |
| Expert: I lead DS/ML research projects or mentor others | −0.2 | −0.2 |
| Not applicable | 0 | 0 |

*Rationale*: more competent → narrower scope desired *and* more user-driven.
Experts know which subset of work is worth automating and have an internal
model of how the pipeline should run, so they want narrow precise help, not
broad system agency. Magnitudes are small because Q3, Q4, Q11 already capture
this signal partially; we are not double-counting.

### Q2 — frequency of AI tool use

| Selection | Scope offset | Initiative offset |
|---|---|---|
| Daily | +0.1 | +0.3 |
| A few times per week | 0 | 0 |
| A few times per month | −0.1 | −0.3 |
| Rarely | −0.2 | −0.5 |
| Never | 0 | 0 |

*Rationale*: heavy users have integrated AI into their workflow deeply enough
to tolerate broader scope and more system-initiated agency; occasional users
want narrow user-driven help on specific moments.

## 4. Weighted-contributor questions

### Q3 — current uses of AI tools (scope axis)

Multi-select. Each option contributes a scope weight; the per-respondent Q3
score is the *mean* of the weights of selected options.

| Weight | Options |
|---|---|
| 1 (narrow) | Coding / debugging · Writing or editing text · Making figures / visualizations through coding · Generate diagrams or illustrations without coding · Managing project notes / documentation · Data cleaning or wrangling |
| 2 (broad) | Literature search / paper summarization · Brainstorming research ideas · Exploratory data analysis · Interpreting results · Model implementation · "I do not currently use AI tools for research" *(sentinel; neutral)* · Other (write-in) |
| 3 (whole task) | Experiment design / ablation planning · Reproducing papers or codebases · Grant / manuscript / reviewer-response support |

**Q3 count modifier** (applies to both axes). Selecting many Q3 options
indicates broad engagement with AI even when the per-option weights are not
themselves broad. This is captured by an additional offset:
- scope: `+0.04 × (n − 3)`, capped at ±0.20
- initiative: `+0.05 × (n − 3)`, capped at ±0.25

where `n` is the number of Q3 options selected.

### Q4 — current frustrations (initiative axis)

Multi-select. Each option contributes an initiative pull; the per-respondent
Q4 contribution is the *sum* of selected pulls.

| Pull | Options | Reading |
|---|---|---|
| −1 | Overconfident answers despite uncertainty · Inaccurate citations, papers, or literature claims · Misunderstanding my dataset, variables, or project context · Suggesting analyses that are technically possible but scientifically inappropriate · Producing outputs that are hard to verify, maintain, or reproduce · Making up files, variables, functions, results, or other project details | the AI took initiative *and was wrong* — push to user-init |
| 0 | Losing context or repeating the same mistake (after correction) · Incorrect DS/ML, statistical, or evaluation reasoning · Coding errors or incorrect package/API usage · Poor handling of compute, environment, or dependency issues · "I do not use general-purpose AI tools" *(sentinel)* · Other | operational pain; doesn't speak to who should drive |
| +1 | I rarely need to correct or work around major issues | implicit trust in the system |

Q4 is structurally biased toward user-init pulls — six options weight −1,
one weights +1. This is honest given that the question asks about
*frustrations*: more frustration with autonomous behavior reasonably implies
wanting more user control. The bias is a feature, not a bug.

### Q10 — what would justify setup burden (both axes)

Multi-select. Each option contributes both a scope weight (averaged with Q3)
and an initiative pull (summed). This is the routing's most consequential
question — every cell has at least one Q10 option mapping directly to it.

| Option | Scope | Initiative | Cell mapped to |
|---|---|---|---|
| Saves substantial time on routine coding/debugging | 1 | −1 | I — Magician |
| Catches mistakes I would otherwise miss | 1 | +2 | III — Justice |
| Understands my dataset/project better than a generic LLM | 2 | +1 | VI — Wheel |
| Helps me produce more reproducible research | 2.5 | +1 | VI — Wheel / VIII — Priestess |
| Helps plan better experiments or ablations | 2.5 | +1 | VI — Wheel / VIII — Priestess |
| Integrates literature, code, and data in one workflow | 3 | +2 | IX — Hermit |
| Lowers my cognitive load across the whole project | 3 | +2 | IX — Hermit |
| Strong word of mouth from trusted researchers | 2 | 0 | (orthogonal — social proof) |
| Nothing; I prefer lightweight generic tools | 1 | −3 | I — Magician (strongest single pull) |
| Other (write-in) | 2 | 0 | (orthogonal) |

### Q11 — top 3 desired qualities (initiative axis)

Multi-select capped at 3. Each option contributes an initiative pull; the
per-respondent Q11 contribution is the *sum* of selected pulls.

| Pull | Options |
|---|---|
| −2 | Human control / approval checkpoints |
| −1 | Clear explanation of uncertainty · Fast implementation |
| 0 | Correctness · Code quality · Privacy/security · Ease of setup · Integration with notebooks/GitHub/cloud/cluster · Recommendation from trusted researchers |
| +1 | Citations/provenance · Dataset understanding · Reproducibility · Ability to recover from failure |
| +2 | Experiment tracking |

Six of fourteen Q11 options weight 0. These are qualities genuinely
orthogonal to initiative (universal goods like correctness, or matters of
fit like setup ease). A respondent whose top 3 are all weight-0 contributes
nothing to initiative from Q11; the routing for them rests on Q4, Q10, and
the modulators.

## 5. Bucket boundaries

The continuous scope and initiative scores are bucketed into discrete cells:

- **Scope buckets**: `< 1.7` → narrow · `1.7 to 2.0` → broad · `≥ 2.0` → whole task
- **Initiative buckets**: `≤ −1.0` → user-initiated · `−1.0 to +1.0` → mixed · `≥ +1.0` → system-initiated

Boundaries were tightened in v0.5 (2026-05-11) from broad 1.7–2.3 and mixed
±1.5. The original ranges produced an over-large residual cell (Lovers,
broad+mixed) that absorbed any respondent with mid-range scores on either
axis. The tightened ranges shrink Lovers' area in score space to roughly
one-third of the prior version and push high-scope respondents into the
whole-task row, which was empty in the original pilot. Sensitivity flagging
in `routing_audit.py` marks respondents within 0.10 of a scope boundary or
0.5 of an initiative boundary as unstable.

## 6. Aggregation

For each respondent the final routing computation is:

```
scope_raw = mean(
    mean(Q3_SCOPE_WEIGHT[item] for item in Q3_selections),
    mean(Q10_SCOPE_WEIGHT[item] for item in Q10_selections),
)
       + Q1C_SCOPE_OFFSET[Q1c]
       + Q2_SCOPE_OFFSET[Q2]
       + clip(0.04 × (len(Q3) − 3), ±0.20)

init_raw  = sum(Q11_INIT_WEIGHT[item] for item in Q11_selections)
       + sum(Q4_INIT_WEIGHT[item] for item in Q4_selections)
       + sum(Q10_INIT_WEIGHT[item] for item in Q10_selections)
       + Q1C_INIT_OFFSET[Q1c]
       + Q2_INIT_OFFSET[Q2]
       + clip(0.05 × (len(Q3) − 3), ±0.25)

scope_bucket = bucket(scope_raw, [1.7, 2.3])
init_bucket  = bucket(init_raw, [-1.5, 1.5])
card         = CARDS[(scope_bucket, init_bucket)]
```

Items not in the weight tables are treated as absent (skipped from
averaging or summing). Respondents who select no Q3 or Q10 options receive
the corresponding axis's default of 2.0 (broad) before modulators apply.

## 7. Known limitations and design choices

These are the routing's known weak points. Each is a deliberate choice with
trade-offs.

1. **Q11 has six weight-0 options.** A respondent picking only these
   contributes nothing from Q11. Defensible because the options really are
   orthogonal to initiative; mitigatable by tightening the weights, at the
   cost of double-counting.

2. **Q4 is structurally user-init biased.** Six options pull toward
   user-init, one pulls system-init. This reflects an honest asymmetry
   in the question (frustrations imply mistrust) but means Q4 alone rarely
   produces system-init routing.

3. **The whole-task row is hard to reach.** Only Q3 has explicit
   scope-3 options; these get averaged with Q10's scope contributions,
   which are at most 3.0 themselves. Crossing the 2.3 whole-task threshold
   typically requires multiple Q3 weight-3 selections *and* Q10's
   broadest options. The 0/9 pilot count on whole-task cells reflects how
   strict this is.

4. **The Lovers cell (broad / mixed) is residual.** No single question
   option pulls toward this cell directly; respondents land there only when
   their other signals neutralize. Conceptually this matches Lovers as the
   centaur middle ground, but it means the cell is hard to *intentionally*
   target.

5. **Many weights are judgment calls.** Specific values (Q1c ±0.2, Q4
   pull magnitudes, the 1/2/3 scope assignments for ambiguous Q3 items)
   are mine, not validated against external ground truth. Sensitivity to
   small reweighting is reported in the audit output.

## 8. Reproducibility

The routing is implemented identically in two places:

- **Python** (`route_responses.py`): used for offline analysis of CSV
  exports and for generating the per-respondent audit document at
  `routing_audit.md`. Run as: `python route_responses.py`
  followed by `python routing_audit.py`.
- **JavaScript** (`qualtrics_routing.js`): runs in the respondent's
  browser at the end of the Qualtrics survey, sets the embedded data
  field `card_slug`, and triggers the End-of-Survey redirect to the
  appropriate card page on the static site.

Both implementations operate on the canonical option strings as exported
by Qualtrics. A longest-prefix-match parser handles multi-select options
that contain internal commas (Q4's "Inaccurate citations, papers, or
literature claims" and similar).

## 9. References

- Allen, J. F. (1999). *Mixed-initiative interaction.* IEEE Intelligent Systems.
- Horvitz, E. (1999). *Principles of mixed-initiative user interfaces.* CHI '99.
- Sheridan, T. B., & Verplank, W. L. (1978). *Human and computer control of undersea teleoperators.* MIT.
- Parasuraman, R., Sheridan, T. B., & Wickens, C. D. (2000). *A model for types and levels of human interaction with automation.* IEEE Transactions on Systems, Man, and Cybernetics.
- Mollick, E. (2024). *Co-Intelligence: Living and Working with AI.* Portfolio.
- Hutchins, E. (1995). *Cognition in the Wild.* MIT Press.

## 10. Versioning

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-05-04 | Initial draft (Q3, Q4, Q11 only). |
| 0.2 | 2026-05-05 | Added Q10, Q1c, Q2; Q11 citations/provenance from 0 to +1. |
| 0.3 | 2026-05-06 | Q1c sign inverted (more competence → narrower + more user-init). Added Q3 count modifier. Added Q1c initiative offset (small symmetric). |
| 0.4 | 2026-05-06 | Reconciled options against live Qualtrics survey (split figure-making, added "Managing project notes," added new Q4/Q11 options, updated Q1c and Q2 wording). |
| 0.5 | 2026-05-11 | Bucket thresholds tightened (broad 1.7–2.0, mixed ±1.0) to shrink Lovers residual cell. |
| **1.0** | **(pending)** | **Lock for pre-registration before opening the expanded survey.** |
