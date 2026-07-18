# cloud-itonami-isco-8212

Open Occupation Blueprint for **ISCO-08 8212**: Electrical and Electronic
Equipment Assemblers.

This repository designs a forkable OSS business for an electrical/
electronic equipment assembly-line scheduling and logistics coordination
practice: a line scheduling and supply-coordination robot manages crew/
task records under a governor-gated actor, so an electrical/electronic
equipment assembly crew keeps its own operating records instead of
renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/elecassemblycoord/` implements the
`ElecAssemblyCoordActor` as a `langgraph.graph/state-graph`
(`elecassemblycoord.actor`) wired to an `Electrical/Electronic Assembly
Line Scheduling Coordination Advisor` (`elecassemblycoord.advisor`) and
an independent `ElecAssemblyCoordGovernor` (`elecassemblycoord.governor`),
following the itonami actor pattern (ADR-2607121000): `:intake ->
:advise -> :govern -> :decide -+-> :commit (:ok? true) +->
:request-approval (:escalate? true, human-in-the-loop interrupt) +->
:hold (:hard? true)`. HARD invariants (always hold, never overridable):
worker provenance, line provenance, no-actuation (`:effect` must be
`:propose`), a closed op-allowlist (`:log-work-record`,
`:schedule-crew-operation`, `:flag-safety-concern`,
`:coordinate-supply-order` — nothing else may ever be proposed), and a
permanent, unconditional block on any proposal that would directly
finalize an assembly-execution decision (e.g. deciding to proceed with a
specific circuit-board assembly run) or a line-safety-clearance decision
(e.g. declaring an assembly line or an ESD-sensitive workstation safe for
handling), or that would override a plant safety officer's judgment.
Always-escalate paths (human sign-off regardless of confidence, mapping
this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above the
registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a line scheduling/logistics coordination
robot performs crew scheduling, production-run/inventory/progress-record
logging and electronic-components-stock supply-order coordination for an
electrical/electronic equipment assembly crew, under an actor that
proposes actions and an independent **Electrical/Electronic Assembly
Line Scheduling Coordination Governor** that gates them. The governor
never dispatches hardware itself, never performs assembly work on the
production line, and never finalizes an assembly-execution decision or a
line-safety-clearance decision, and never overrides a plant safety
officer's judgment; `:high`/`:safety-critical` actions (such as a
flagged electrical-shock/ESD-condition/equipment-condition concern, or
an above-threshold supply order) require human sign-off. **This actor
coordinates LINE SCHEDULING/LOGISTICS ONLY — it never performs assembly
work itself, and it never makes a line-safety-clearance decision
itself.**

Electrical and Electronic Equipment Assemblers assemble circuit boards
and electrical equipment on production lines, presenting both an
electrical-shock hazard and an ESD (electrostatic discharge)-sensitive
component-handling hazard. This is a real electrical-hazard and
component-damage domain; this actor never assembles that equipment and
never clears it as safe — it only schedules and logs around it, and
always routes electrical-hazard/ESD-condition/safety concerns to a
human plant safety officer.

## Core Contract

```text
crew roster + line registration + safety-reporting policy
        |
        v
Electrical/Electronic Assembly Line Scheduling Coordination Advisor -> ElecAssemblyCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize an assembly-execution decision, finalize a line-safety-clearance
decision, override a plant safety officer's judgment, suppress an
operating record, or disclose sensitive data without governor approval
and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `8212`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
