# Workflow Automation Patterns

Workflow automation patterns are reusable architectural solutions to common automation design problems. SmartAgents engineers apply these patterns consistently across client deployments to ensure reliability, maintainability, and scalability.

---

## Core Patterns

### 1. Sequential Flow Pattern

**Problem:** A business process requires steps to execute in strict order, where each step depends on the output of the previous step.

**Solution:** A linear automation chain where each node receives the output of the previous node as its input. The flow terminates on success or routes to error handling on failure at any step.

**Use cases:** Invoice generation, client onboarding, order processing, document generation.

**Key design considerations:**
- Each step must validate its input before processing
- Failures at any step must not silently continue to the next step
- The final step must confirm successful delivery to the output system

---

### 2. Parallel Branch Pattern

**Problem:** Multiple independent actions need to execute simultaneously after a trigger event, and the flow should continue only after all branches complete.

**Solution:** A fan-out node distributes execution to multiple parallel branches. A merge node waits for all branches to complete before continuing.

**Use cases:** Multi-channel notifications (WhatsApp + email + Slack simultaneously), simultaneous CRM and accounting updates, parallel approval workflows.

**Key design considerations:**
- Branches must be genuinely independent (no shared mutable state)
- The merge node must handle partial failures gracefully
- Timeout limits must be set on branches to prevent indefinite blocking

---

### 3. Conditional Routing Pattern

**Problem:** The workflow path must differ based on data conditions evaluated at runtime.

**Solution:** A decision node evaluates conditions against the current data context and routes execution to the appropriate branch.

**Use cases:** Lead scoring routing (high score → immediate contact, low score → nurture sequence), approval escalation (above threshold → manager, below → auto-approve), tiered response systems.

**Key design considerations:**
- Conditions must be explicit and exhaustive (every possible value must route somewhere)
- A default route must exist for unexpected values
- Condition logic must be documented clearly for maintainability

---

### 4. Wait and Resume Pattern

**Problem:** A workflow must pause at a specific step until an external event occurs before continuing.

**Solution:** The workflow enters a waiting state and registers a listener for the expected event. When the event occurs, the workflow resumes from the paused step with updated context.

**Use cases:** Payment confirmation waiting (trigger invoice delivery, wait for payment, then trigger onboarding), document collection (request documents, wait for upload confirmation, then proceed), subscription renewal (trigger 14 days before expiry, wait for renewal or cancellation).

**Key design considerations:**
- Maximum wait duration must be defined with explicit timeout handling
- Reminder triggers can be nested within the wait state
- The resumed workflow must re-validate context in case conditions changed during the wait

---

### 5. Aggregation Pattern

**Problem:** Data from multiple sources must be collected and combined before a downstream step can execute.

**Solution:** An aggregation node collects results from multiple data sources (via API calls, database queries, or sub-flows) and merges them into a unified data object.

**Use cases:** Consolidated operational reports (CRM + accounting + ads data), multi-system dashboards, batch invoice runs, combined lead scoring (profile data + behavioural data + external enrichment).

**Key design considerations:**
- Partial failures (one source unavailable) must be handled explicitly
- Data freshness requirements must be defined (is 1-hour-old data acceptable?)
- Merge logic must handle schema differences between sources

---

### 6. Event Deduplication Pattern

**Problem:** The same trigger event may be received multiple times due to network retries, platform bugs, or user actions, causing duplicate automation executions.

**Solution:** Each incoming event is assigned a unique idempotency key. Before processing, the system checks whether this key has been processed previously. If yes, the event is acknowledged and discarded without re-processing.

**Use cases:** Webhook handlers, payment event processors, form submission handlers.

**Key design considerations:**
- Idempotency keys must be derived from the event content (not assigned randomly)
- Key storage must be fast (cache-based) and have appropriate TTL
- Deduplication must occur before any external side effects are triggered

---

## Pattern Combinations

Complex automation systems typically combine multiple patterns.

**Example: Lead Capture + Qualification + Routing**

```
Webhook trigger (form submission)
    ↓
Event deduplication check
    ↓
Sequential: validate → enrich → score
    ↓
Conditional routing: score threshold
    ├── High score → parallel: (CRM tag + WhatsApp notification + task creation)
    └── Low score → sequential: (CRM tag + add to nurture sequence)
```

This single lead flow combines sequential, deduplication, conditional routing, and parallel branch patterns.

---

## Related Documentation

- [Automation Architecture](./automation-architecture.md)
- [Operational Flow Design](./operational-flow-design.md)
- [CRM Automation Integration](./crm-automation-integration.md)

🌐 https://SmartAgents.ma
