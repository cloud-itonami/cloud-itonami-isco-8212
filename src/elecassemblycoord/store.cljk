(ns elecassemblycoord.store
  "SSoT for the ISCO-08 8212 electrical and electronic equipment
  assemblers assembly-line scheduling/logistics coordination actor
  (itonami actor pattern, ADR-2607121000 / CLAUDE.md Actors section;
  README's 'Robotics premise' — a line scheduling/logistics
  coordination robot performs crew scheduling, production-run/
  inventory/progress-record logging and electronic-components-stock
  supply-order coordination for an electrical/electronic equipment
  assembly crew under this advisor/governor pair, which never
  dispatches hardware itself, never performs assembly work itself, and
  never finalizes an assembly-execution decision or a line-safety-
  clearance decision, and never overrides a plant safety officer's
  judgment — those remain the plant safety officer's exclusive
  judgment). Modeled closely on cloud-itonami-isco-8122's
  platingcoord.store.

  Domain:

    worker — a registered electrical/electronic equipment assembler
             crew member (:worker-id, :name)
    line   — a registered assembly line {:line-id :name
             :max-supply-cost number}. `:max-supply-cost` is an
             informational registered ceiling used only to decide
             whether a `:coordinate-supply-order` proposal escalates
             to human sign-off (the governor never blocks a
             within-threshold order outright; it only decides commit
             vs. escalate).
    record — a committed operating record (a logged production-run/
             inventory/progress entry, a scheduled crew/shift
             operation, a flagged safety concern, or a coordinated
             electronic-components-stock supply order) — written ONLY
             via commit-record!. This actor coordinates line
             scheduling/logistics ONLY — a `record` is a coordination
             artifact, never an assembly-execution act, never a
             line-safety-clearance decision, and never a plant safety
             officer's-judgment override.
    ledger — append-only audit trail, commit or hold.")

(defprotocol Store
  (worker [s worker-id])
  (line [s line-id])
  (records-of [s worker-id])
  (ledger [s])
  (register-worker! [s worker])
  (register-line! [s line])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (line [_ line-id] (get-in @a [:lines line-id]))
  (records-of [_ worker-id] (filter #(= worker-id (:worker-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-worker! [s w]
    (swap! a assoc-in [:workers (:worker-id w)] w) s)
  (register-line! [s l]
    (swap! a assoc-in [:lines (:line-id l)] l) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:workers {} :lines {} :records [] :ledger []}
                                    seed)))))
