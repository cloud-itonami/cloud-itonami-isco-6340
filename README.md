# cloud-itonami-isco-6340

Open Occupation Blueprint for **ISCO-08 6340**: Subsistence Fishers,
Hunters, Trappers and Gatherers.

This repository designs a forkable OSS business for subsistence
fishing/hunting/trapping/gathering logistics coordination: a
logistics-coordination robot manages catch/harvest records,
equipment-maintenance schedules and non-weapon-equipment procurement
paperwork under a governor-gated actor, so a subsistence household
keeps its own operating records instead of renting a closed
wildlife/resource-operations SaaS.

**This actor has no weapon-firing, trap-setting/springing, fishing/
hunting/gathering-execution or kill/harvest-timing-decision authority,
structurally, not merely by policy gate.** Real subsistence fishers,
hunters, trappers and gatherers exercise gear/weapons/traps and
directly take fish, game or wild plant/animal resources for household
consumption — that is their domain's actual physical work, combining
the weapons/kill-decision concerns of hunting/trapping with the
livelihood-vulnerability dimension of subsistence agriculture. This
actor is a logistics/record-keeping coordination robot ONLY. Its
op-allowlist has exactly four ops — `:log-harvest-record`,
`:schedule-equipment-operation`, `:flag-livelihood-concern`,
`:coordinate-supply-order` — and no op resembling firing a weapon,
setting or springing a trap, making a fishing/hunting/gathering-
execution decision, or making a kill/harvest-timing decision exists
anywhere in the codebase. That absence is enforced twice, independently:
a closed op-allowlist (with a second, independently-named
scope-excluded-op list as defense-in-depth) that hard-blocks any op
outside the four above, and a defense-in-depth text scan that
hard-blocks any proposal whose rationale claims to finalize or execute
one of those actions. See
[`src/subsistenceharvest/governor.cljc`](src/subsistenceharvest/governor.cljc)'s
namespace docstring for the full structural argument. Any observation
that MAY warrant food-security, resource-depletion or safety attention
is surfaced ONLY via `:flag-livelihood-concern`, which always escalates
immediately to the human household member and is never
auto-commit-eligible — the robot never acts on it itself, not even as
a `:propose`. Area use, season, and catch/bag-limit decisions belong
to the human household member and any applicable regulatory/customary
authority, never this actor.

**Maturity: `:implemented`.** `src/subsistenceharvest/` implements the
`SubsistenceHarvestActor` as a `langgraph.graph/state-graph`
(`subsistenceharvest.actor`) wired to a `Subsistence Harvest Advisor`
(`subsistenceharvest.advisor`) and an independent
`SubsistenceHarvestGovernor` (`subsistenceharvest.governor`), following
the itonami actor pattern (ADR-2607011000 / ADR-2607121000): `:intake
-> :advise -> :govern -> :decide -+-> :commit (:ok?) +->
:request-approval (:escalate?, human-in-the-loop interrupt) +-> :hold
(:hard?)`. HARD invariants (always hold, never overridable): operator
provenance, area provenance, no-actuation (`:effect` must be
`:propose`), a closed op-allowlist (plus a second, independently-named
scope-excluded-op list) with no op resembling weapon-firing/
trap-setting/fishing-hunting-gathering-execution/kill decisions, a
defense-in-depth text scan blocking any rationale that finalizes or
executes such an action, a registered area basis for any area-scoped
proposal, and an attached harvest report before any harvest record can
be logged (logging a record without one is a fabricated record, not
documentation). Always-escalate ops (human sign-off regardless of
confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-livelihood-concern` (always, never auto-commit-eligible) and
`:coordinate-supply-order` above the area's registered supply-order
cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a
**robot performs the physical/administrative domain work**. Here a
logistics-coordination robot performs catch/harvest record entry
(post-hoc metadata only, never a kill/harvest-timing decision),
equipment-maintenance scheduling and non-weapon equipment/supplies
procurement paperwork under an actor that proposes actions and an
independent **Subsistence Harvest Governor** that gates them. The
governor never dispatches hardware itself and never fires a weapon,
sets or springs a trap, makes a fishing/hunting/gathering-execution
decision, or makes a kill/harvest-timing decision —
`:flag-livelihood-concern` actions always require human household-
member sign-off, and no op resembling those execution actions exists
in the allowlist at all.

## Core Contract

```text
subsistence household request (harvest log / maintenance / procurement / concern)
        |
        v
Subsistence Harvest Advisor -> Subsistence Harvest Governor -> log/schedule/order, or human sign-off
        |
        v
robot actions (gated, documentation/logistics only) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
suppress an operating record, or exercise any weapon-firing,
trap-setting/springing, fishing/hunting/gathering-execution or
kill/harvest-timing-decision authority — that authority does not exist
in this actor's vocabulary.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `6340`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
