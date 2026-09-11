(ns floortile.store
  "SSoT for the ISCO-08 7122 floor-laying/tile-setting job-site
  scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a job-site scheduling/logistics coordination robot
  performs crew scheduling, task/materials-usage/progress-record
  logging and flooring/tile-materials supply-order coordination for a
  floor-laying/tile-setting crew under this advisor/governor pair,
  which never dispatches hardware itself, never performs
  flooring/tile-installation work itself, and never finalizes a
  flooring/tile-installation-execution decision or overrides a site
  safety officer's judgment — those remain a site safety officer's
  exclusive judgment). Modeled on cloud-itonami-isco-7111's
  housebuilder.store (job-site scheduling/logistics coordination
  shape) and cloud-itonami-isco-3313's accountingsupport.store.

  Domain:

    installer — a registered floor-layer/tile-setter crew member
                (:installer-id, :name)
    site      — a registered job site {:site-id :name
                :max-supply-cost number}. `:max-supply-cost` is an
                informational registered ceiling used only to decide
                whether a `:coordinate-supply-order` proposal
                escalates to human sign-off (the governor never
                blocks a within-threshold order outright; it only
                decides commit vs. escalate).
    record    — a committed operating record (a logged
                task/materials-usage/progress entry, a scheduled crew
                operation, a flagged safety concern, or a coordinated
                flooring/tile-materials supply order) — written ONLY
                via commit-record!.
    ledger    — append-only audit trail, commit or hold.")

(defprotocol Store
  (installer [s installer-id])
  (site [s site-id])
  (records-of [s installer-id])
  (ledger [s])
  (register-installer! [s installer])
  (register-site! [s st])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (installer [_ installer-id] (get-in @a [:installers installer-id]))
  (site [_ site-id] (get-in @a [:sites site-id]))
  (records-of [_ installer-id] (filter #(= installer-id (:installer-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-installer! [s i]
    (swap! a assoc-in [:installers (:installer-id i)] i) s)
  (register-site! [s st]
    (swap! a assoc-in [:sites (:site-id st)] st) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:installers {} :sites {} :records [] :ledger []}
                                    seed)))))
