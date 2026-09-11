(ns subsistenceharvest.governor
  "SubsistenceHarvestGovernor — the independent safety/traceability
  layer gating every logistics-coordination proposal an advisor may
  make for a subsistence fisher/hunter/trapper/gatherer. The governor
  never dispatches hardware itself and never exercises, simulates
  exercising, or approves exercising ANY decision to fire a weapon,
  set/spring a trap, make a fishing/hunting/gathering-execution
  decision, or make a kill/harvest-timing decision. Modeled on
  cloud-itonami-isco-6224's huntingtrapping.governor (weapon/trap/
  kill-decision exclusion pattern) and adapted for
  cloud-itonami-isco-6310's subsistencefarm.governor livelihood-
  vulnerability dimension.

  == Why this actor cannot become a weapon-firing / trap-setting /
     fishing-hunting-gathering-execution / kill-decision authority
     (structural, not merely gated) ==

  Real subsistence fishers, hunters, trappers and gatherers exercise
  gear/weapons/traps and directly take fish, game or wild plant/animal
  resources — that is their domain's actual physical work, combining
  the weapons/kill-decision concerns of hunting/trapping with the
  livelihood-vulnerability dimension of subsistence agriculture: this
  harvest is typically for direct household consumption, so a
  resource-depletion or food-security risk threatens the household's
  own food supply directly, not merely a commercial outcome. This
  actor is a logistics/record-keeping coordination robot ONLY — it
  helps log catch/harvest records (post-hoc metadata, never a
  kill/harvest-timing decision), schedule equipment maintenance, and
  surface food-security/resource-depletion/safety concerns (never
  decides on area use, season, or catch/bag limits itself; those
  determinations belong to the human household member and any
  applicable regulatory/customary authority). It is deliberately built
  so that weapon-firing/trap-setting/fishing-hunting-gathering-
  execution/kill-decision authority can never leak in through two
  independent layers:

    Layer 1 — closed op-allowlist (`op-allowlist`, rule :unknown-op).
      The only ops this governor will ever accept are
      `:log-harvest-record`, `:schedule-equipment-operation`,
      `:flag-livelihood-concern` and `:coordinate-supply-order`. No op
      resembling firing a weapon, setting/springing a trap, making a
      fishing/hunting/gathering-execution decision, or making a
      kill/harvest-timing decision exists anywhere in this codebase's
      vocabulary — not as a gated/escalated op, not as a `:hold`-by-
      default op, not at all. Any op keyword outside this four-op set
      — including a hypothetical `:fire-weapon` or `:spring-trap` a
      buggy or malicious caller might construct directly, bypassing
      the advisor entirely — is unconditionally hard-blocked here. A
      second, independent HARD check (`scope-excluded-ops`) names ten
      concretely-forbidden ops as defense-in-depth so a future
      allowlist edit cannot silently re-open this specific out-of-
      scope path without also touching this list. This is the primary
      structural guarantee: the allowlist has nothing to point at.

    Layer 2 — defense-in-depth textual scope-exclusion
      (`subsistence-execution-scope-exclusion-phrases`, rule
      :subsistence-execution-scope-exclusion). Even though no op can
      express a weapon-firing/trap-setting/fishing-hunting-gathering-
      execution/kill-decision action, this governor also hard-blocks
      any proposal whose free-text `:rationale` claims to finalize or
      execute one of those actions (\"fire the weapon\", \"spring the
      trap\", \"set the trap\", \"make the fishing execution
      decision\", \"make the hunting execution decision\", \"make the
      gathering execution decision\", \"make the trapping execution
      decision\", \"make the kill decision\", \"make the
      harvest-timing decision\"). This is deliberately phrased as
      finalization/execution ACTION PHRASES, never bare nouns like
      \"weapon\", \"trap\", \"fish\", \"hunt\", \"gather\" or \"kill\"
      — a bare-noun list would false-trip on completely routine
      documentation rationale (e.g. this actor's own default
      mock-advisor text describing that NO weapon-firing action is
      proposed necessarily mentions the word \"weapon\"). See
      advisor.cljc's docstring and the
      `default-mock-advisor-proposals-never-self-trip-scope-exclusion`
      test in governor_test.clj, which pins this down for every op the
      mock advisor can produce.

  Any observation this actor's robot logs that MAY warrant human
  attention (food-security, resource-depletion, safety, or area/
  season/catch-limit questions) is surfaced ONLY via the
  always-escalating `:flag-livelihood-concern` op — reviewed and acted
  on by the human household member themselves. The robot never acts on
  it, not even as a `:propose`.

  HARD invariants (:hard? true, ALWAYS :hold, permanent,
  un-overridable):
    1. operator provenance       — the household member must be
                                    independently verified/registered.
    2. area provenance           — the fishing/hunting/trapping/
                                    gathering area must be
                                    independently verified/registered.
    3. no-actuation               — proposal :effect must be :propose
                                    (the governor never dispatches
                                    hardware and never itself fires a
                                    weapon, sets/springs a trap, makes
                                    a fishing/hunting/gathering-
                                    execution decision, or makes a
                                    kill/harvest-timing decision; it
                                    only gates what the advisor may
                                    log/schedule/flag/order).
    4. closed op-allowlist        — see Layer 1 above (:unknown-op).
    5. scope-excluded op          — see Layer 1's second independent
                                    check above
                                    (:subsistence-execution-scope-
                                    excluded-op).
    6. subsistence-execution scope exclusion — see Layer 2 above
                                    (:subsistence-execution-scope-
                                    exclusion).
    7. area basis                 — an area-scoped proposal must cite
                                    a REGISTERED area belonging to this
                                    operator.
    8. harvest report attached    — a `:log-harvest-record` proposal
                                    must have
                                    `:harvest-report-attached?` true
                                    (a post-hoc field report/log basis)
                                    before it can be logged (logging a
                                    record with no attached harvest
                                    report is a fabricated record, not
                                    documentation).
  ESCALATION invariants (:escalate? true, ALWAYS human household-
  member sign-off — these are :high/:safety-critical regardless of
  confidence):
    9. :op :flag-livelihood-concern — always escalates immediately and
       is never auto-commit-eligible; the human household member
       reviews and decides whether any further action (which this
       actor cannot itself take) is warranted.
   10. :coordinate-supply-order whose :cost exceeds the area's
       registered `:max-supply-order-cost`.
   11. low confidence (< `confidence-floor`)."
  (:require [subsistenceharvest.store :as store]
            [kotoba.lang.text :as str]))

(def confidence-floor 0.6)

(def ^:private op-allowlist
  #{:log-harvest-record :schedule-equipment-operation
    :flag-livelihood-concern :coordinate-supply-order})

(def ^:private always-escalate-ops #{:flag-livelihood-concern})

;; Defense-in-depth: none of these ops are ever in `op-allowlist`
;; above, so they are already refused by the closed-allowlist check
;; below; they are named again here — as explicit finalization/
;; execution ACTIONS, never bare nouns — so a future allowlist edit
;; cannot silently re-open this specific out-of-scope path without
;; also touching this list.
(def ^:private scope-excluded-ops
  #{:fire-weapon :discharge-weapon
    :set-trap :spring-trap
    :make-kill-decision :make-harvest-timing-decision
    :make-fishing-execution-decision :make-hunting-execution-decision
    :make-gathering-execution-decision :make-trapping-execution-decision})

;; Deliberately full finalization/execution ACTION PHRASES, never bare
;; nouns — see the namespace docstring's "Layer 2" explanation and the
;; self-tripping-bug guardrail this exists to avoid.
(def ^:private subsistence-execution-scope-exclusion-phrases
  ["fire the weapon" "fired the weapon" "fires the weapon" "firing the weapon"
   "fire a weapon" "fired a weapon" "fires a weapon" "firing a weapon"
   "discharge the weapon" "discharged the weapon" "discharges the weapon" "discharging the weapon"
   "discharge a weapon" "discharged a weapon" "discharges a weapon" "discharging a weapon"
   "set the trap" "sets the trap" "setting the trap"
   "set a trap" "sets a trap" "setting a trap"
   "spring the trap" "sprang the trap" "springs the trap" "springing the trap" "sprung the trap"
   "spring a trap" "sprang a trap" "springs a trap" "springing a trap"
   "make the kill decision" "made the kill decision" "makes the kill decision" "making the kill decision"
   "make the harvest-timing decision" "made the harvest-timing decision"
   "makes the harvest-timing decision" "making the harvest-timing decision"
   "make the fishing execution decision" "made the fishing execution decision"
   "makes the fishing execution decision" "making the fishing execution decision"
   "make the hunting execution decision" "made the hunting execution decision"
   "makes the hunting execution decision" "making the hunting execution decision"
   "make the gathering execution decision" "made the gathering execution decision"
   "makes the gathering execution decision" "making the gathering execution decision"
   "make the trapping execution decision" "made the trapping execution decision"
   "makes the trapping execution decision" "making the trapping execution decision"])

(defn- scope-excluded-text? [s]
  (let [s (str/lower (or s ""))]
    (boolean (some #(str/includes? s %) subsistence-execution-scope-exclusion-phrases))))

(defn- needs-area? [op]
  (contains? op-allowlist op))

(defn- hard-violations [{:keys [request proposal]} operator-record a]
  (let [{:keys [op area-id harvest-report-attached? rationale]} proposal
        log? (= :log-harvest-record op)]
    (cond-> []
      (nil? operator-record)
      (conj {:rule :no-operator :detail "未登録 operator — provenance が独立に検証されていない"})

      (not= :propose (:effect proposal))
      (conj {:rule :no-actuation
             :detail "effect は :propose のみ許可（governor は発砲・罠の設置/作動・漁猟採集の実行を一切直接実行しない）"})

      (not (contains? op-allowlist op))
      (conj {:rule :unknown-op
             :detail (str "許可されていない op: " (pr-str op)
                          "（発砲・罠の設置/作動・漁猟採集の実行・捕殺/収穫タイミングの判断に"
                          "相当する op は closed allowlist に一切存在しない）")})

      (contains? scope-excluded-ops op)
      (conj {:rule :subsistence-execution-scope-excluded-op
             :detail (str "op " (pr-str op) " は発砲・罠の設置/作動・漁猟採集の実行・"
                          "捕殺/収穫タイミングの判断に相当し、恒久的に禁止")})

      (scope-excluded-text? rationale)
      (conj {:rule :subsistence-execution-scope-exclusion
             :detail (str "提案文言が発砲・罠の設置/作動・漁猟採集の実行・捕殺/収穫タイミングの判断を"
                          "確定・実行する記述を含む — たとえ :propose であっても恒久的に禁止"
                          "（allowlistに存在しない権限を文言で回避することを防ぐ多層防御）")})

      (and (needs-area? op) (nil? area-id))
      (conj {:rule :missing-area :detail "area-id が未指定"})

      (and (needs-area? op) area-id (nil? a))
      (conj {:rule :unknown-area :detail "未登録 area への提案は不可"})

      (and (needs-area? op) area-id a
           (not= (:operator-id a) (:operator-id request)))
      (conj {:rule :area-wrong-operator :detail "area が別 operator の管轄"})

      (and log? (not harvest-report-attached?))
      (conj {:rule :missing-harvest-report
             :detail "原始記録（フィールドレポート等）が添付されていない記録の記入は捏造記録であって文書化ではない"}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `subsistenceharvest.store/Store`. Pure — never
  mutates the store, never fires a weapon, sets/springs a trap, makes
  a fishing/hunting/gathering-execution decision, or makes a
  kill/harvest-timing decision."
  [request context proposal store]
  (let [operator-record (store/operator store (:operator-id request))
        a (some->> (:area-id proposal) (store/area store))
        hard (hard-violations {:request request :proposal proposal} operator-record a)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        over-cost? (and (= :coordinate-supply-order (:op proposal))
                        a (number? (:cost proposal))
                        (number? (:max-supply-order-cost a))
                        (> (:cost proposal) (:max-supply-order-cost a)))
        always-risky? (or (contains? always-escalate-ops (:op proposal)) over-cost?)]
    {:ok? (and (not hard?) (not low?) (not always-risky?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky?))}))
