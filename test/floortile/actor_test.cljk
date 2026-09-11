(ns floortile.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [floortile.actor :as actor]
            [floortile.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-installer! st {:installer-id "installer-1" :name "Kobo Yamada"})
    (store/register-site! st {:site-id "S-1" :name "Kobo Renovation Site" :max-supply-cost 2000})
    st))

(deftest commits-a-registered-work-log
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:installer-id "installer-1" :op :log-work-record :stake :low
                  :site-id "S-1" :task "tile-setting progress log"}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "installer-1"))))))

(deftest holds-an-unregistered-site-proposal
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:installer-id "installer-1" :op :log-work-record :stake :low
                  :site-id "S-ghost" :task "tile-setting progress log"}
        result (actor/run-request! graph request {} "thread-2")]
    (is (= :hold (:disposition (:state result))))
    (is (empty? (store/records-of st "installer-1")))))

(deftest interrupts-then-approves-safety-concern-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:installer-id "installer-1" :op :flag-safety-concern :stake :low
                  :site-id "S-1" :hazard-type :fume-exposure}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "installer-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "installer-1")))))))

(deftest holds-a-scope-excluded-op-even-at-high-confidence
  (testing "an actor run can never commit a proposal that would finalize a flooring/tile-installation-execution decision, regardless of disposition path"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:installer-id "installer-1" :op :finalize-tile-installation-decision :stake :low
                    :site-id "S-1" :task "tile installation decision"}
          result (actor/run-request! graph request {} "thread-4")]
      (is (= :done (:status result)))
      (is (= :hold (:disposition (:state result))))
      (is (empty? (store/records-of st "installer-1"))))))
