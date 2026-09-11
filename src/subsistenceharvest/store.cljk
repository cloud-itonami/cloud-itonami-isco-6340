(ns subsistenceharvest.store
  "SSoT for the ISCO-08 6340 subsistence fishing/hunting/trapping/
  gathering household record-keeping/logistics coordination actor
  (itonami actor pattern, ADR-2607121000 / CLAUDE.md Actors section;
  README's 'Robotics premise' — a logistics/record-keeping
  coordination robot performs catch/harvest record entry (post-hoc
  metadata only), equipment-maintenance scheduling and non-weapon
  supplies procurement paperwork for a subsistence-household member
  under this advisor/governor pair, which never dispatches hardware
  itself and never exercises, simulates exercising, or proposes
  exercising ANY decision to fire a weapon, set/spring a trap, make a
  fishing/hunting/gathering-execution decision, or make a
  kill/harvest-timing decision — that authority is not merely gated,
  it is structurally absent from this actor's op-allowlist; see
  subsistenceharvest.governor). Modeled closely on
  cloud-itonami-isco-6224's huntingtrapping.store (two-layer
  op-allowlist + text-scan exclusion pattern, permit/area provenance
  shape, harvest-report-attachment basis) and adapted for
  cloud-itonami-isco-6310's subsistencefarm.store livelihood-
  vulnerability dimension: this class covers subsistence-level
  fishing, hunting, trapping AND gathering practiced primarily for
  household consumption, so a resource-depletion/food-security risk
  threatens the household's own food supply directly, not merely a
  commercial outcome — `:flag-livelihood-concern` always escalates.

  Domain:

    operator — a registered, independently-verified household member
               who fishes, hunts, traps or gathers for household
               subsistence (:operator-id :name). Provenance must be
               established before any proposal for this operator can
               be considered.
    area     — a registered fishing ground, hunting ground, trapline
               or gathering site {:area-id :operator-id :name
               :max-supply-order-cost}. `:max-supply-order-cost` is
               the registered threshold a proposed
               `:coordinate-supply-order` cost must not exceed without
               escalating to human household sign-off (NOT a hard
               block — procurement above threshold is legitimate,
               routine administrative work that simply requires human
               sign-off, unlike this actor's complete lack of any op
               resembling a weapon-firing, trap-setting/springing,
               fishing/hunting/gathering-execution or
               kill/harvest-timing decision). Resource-management
               (bag/catch limits, gathering-area boundaries, season
               windows) determinations belong to the human household
               member and any applicable regulatory/customary
               authority — this actor never decides on the area's use
               itself, it only cites an area's registration as a
               provenance basis for routing paperwork.
    record   — a committed operating record (a logged harvest entry,
               an equipment-maintenance schedule, a flagged livelihood
               concern, or a coordinated supply order) — written ONLY
               via commit-record!. Never a kill record, a trap-
               deployment record, a fishing/hunting/gathering-
               execution record or a weapon-discharge record — no such
               record type exists because no such op exists.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (operator [s operator-id])
  (area [s area-id])
  (records-of [s operator-id])
  (ledger [s])
  (register-operator! [s op])
  (register-area! [s a])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (operator [_ operator-id] (get-in @a [:operators operator-id]))
  (area [_ area-id] (get-in @a [:areas area-id]))
  (records-of [_ operator-id] (filter #(= operator-id (:operator-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-operator! [s op]
    (swap! a assoc-in [:operators (:operator-id op)] op) s)
  (register-area! [s ar]
    (swap! a assoc-in [:areas (:area-id ar)] ar) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:operators {} :areas {} :records [] :ledger []}
                                    seed)))))
