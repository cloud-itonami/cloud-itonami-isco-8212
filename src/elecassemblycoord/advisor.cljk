(ns elecassemblycoord.advisor
  "Electrical/Electronic Assembly Line Scheduling Coordination Advisor
  — proposing a line scheduling/logistics coordination operation (log
  a work record, schedule a crew operation, flag a safety concern,
  coordinate an electronic-components-stock supply order) from a crew
  roster, line registration and safety-reporting policy. Swappable
  mock/llm; the advisor ONLY proposes — `elecassemblycoord.governor`
  independently gates every proposal and always escalates safety
  concerns and above-threshold supply orders. The advisor never
  proposes to directly finalize an assembly-execution decision (e.g.
  deciding to proceed with a specific circuit-board assembly run), or
  a line-safety-clearance decision (e.g. declaring an assembly line or
  an ESD-sensitive workstation safe for handling), and never proposes
  to override a plant safety officer's judgment — those stay
  permanently out of this actor's scope. Modeled closely on
  cloud-itonami-isco-8122's platingcoord.advisor.

  A proposal: {:op :log-work-record|:schedule-crew-operation|
               :flag-safety-concern|:coordinate-supply-order
               :effect :propose :worker-id str :line-id str
               :cost number :hazard-type kw :task str :stake kw
               :confidence n :rationale str}")

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- rationale-for [op worker-id line-id hazard-type]
  (case op
    :log-work-record
    (str "logged work record for assembler " worker-id " at line " line-id)

    :schedule-crew-operation
    (str "scheduled crew operation for assembly task at line " line-id)

    :flag-safety-concern
    (str "flagged " (name (or hazard-type :hazard)) " concern for assembler "
         worker-id " at line " line-id " — routed for plant safety officer review")

    :coordinate-supply-order
    (str "coordinated supply order for assembler " worker-id " at line " line-id)

    (str "proposed " (name op) " for assembler " worker-id " at line " line-id)))

(defn- infer [_store {:keys [op stake worker-id line-id cost hazard-type task]
                       :as request}]
  {:op op
   :effect :propose
   :worker-id worker-id
   :line-id line-id
   :cost cost
   :hazard-type hazard-type
   :task task
   :stake (or stake :low)
   :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
   :rationale (rationale-for op worker-id line-id hazard-type)})

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are an electrical and electronic equipment assembly line
   scheduling/logistics coordination advisor. Given a request, propose
   an :op (one of :log-work-record, :schedule-crew-operation,
   :flag-safety-concern, :coordinate-supply-order), the :worker-id,
   :line-id, and any :cost/:hazard-type/:task fields, an honest
   :confidence and a :stake. Never propose an op outside this closed
   list, and never propose to directly finalize an assembly-execution
   decision (e.g. deciding to proceed with a specific circuit-board
   assembly run), or a line-safety-clearance decision (e.g. declaring
   an assembly line or an ESD-sensitive workstation safe for
   handling), or to override a plant safety officer's judgment — those
   are always out of this actor's scope; it coordinates line
   scheduling/logistics only and never performs assembly work or
   clears lines/workstations for handling/shipment itself. Safety
   concerns always require human sign-off regardless of confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
