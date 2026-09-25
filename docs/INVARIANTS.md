Core Financial Invariants
INV-01: Conservation of Value

The sum of all balances (escrow + merchant + customer + fees) MUST equalthe total value deposited into the system. No value is created or destroyedoutside of explicit protocol actions.
INV-02: No Unauthorized Withdrawals

Only the designated recipient (merchant on fulfill, customer on refund,arbiter on dispute resolution) may claim payment funds. No third partycan withdraw funds from an escrow they are not a party to.
INV-03: No Overpayment

The total amount released from a payment MUST NOT exceed the originaldeposited amount plus any permitted fee adjustments. Each payment releasesat most its principal.
INV-04: No Double-Settlement

A payment can transition to a terminal state (Fulfilled, Refunded,DisputeResolved) exactly once. No payment can be fulfilled AND refunded,or settled twice.
State Machine Invariants
INV-05: Valid State Transitions Only

The payment state machine only permits:  Created → Funded → Fulfilled  Created → Funded → Disputed → DisputeResolved  Created → Funded → Refunded (after expiry)  Created → Expired (if never funded)

No backward or cross-branch transitions are valid.
INV-06: Expiry Monotonicity

A payment whose expiry timestamp has passed cannot be fulfilled. It canonly be refunded or disputed (if already in dispute).
INV-07: Nonce Uniqueness

No two payments can share the same (creator, nonce) pair. Replay of apreviously consumed nonce MUST be rejected.
INV-08: Authorization Consistency

The actor performing a transition MUST be authorized:

    Only creator can fund
    Only merchant can fulfill
    Only customer can request refund
    Only designated arbiter can resolve dispute

Edge-Case Invariants
INV-09: Zero-Amount Payment

Zero-amount payments follow the same state machine but MUST NOT resultin any token transfers.
INV-10: Fee Ceiling

Protocol fees collected per payment MUST NOT exceed the configuredmaximum fee percentage of the payment amount.
Transaction Submission Invariants
INV-11: Submission Idempotency

A transaction submission MUST be idempotent per (payment, operation, nonce).Re-submitting the same logical operation with the same nonce MUST NOTproduce a second on-chain effect; the orchestrator MUST return the existingresult (or a stable duplicate error) rather than broadcasting again.
INV-12: Confirmation Before Settlement

A payment MUST NOT be treated as settled (Funded, Fulfilled, Refunded,DisputeResolved) until its submission is confirmed on the supported network.Until confirmation, the payment remains in its pre-submission state and anyoptimistic UI state MUST be reconcilable from the confirmed ledger.
INV-13: Bounded Retry

Retries MUST be bounded and MUST only apply to retryable failures (e.g.transient network or timeout errors). Non-retryable failures (validation,authorization, insufficient balance, expired, malformed) MUST NOT be retriedand MUST surface a stable error. Retries MUST reuse the same signed transactionor nonce so that a retry cannot double-spend.
INV-14: Rollback on Terminal Failure

When a submission reaches a terminal failure after exhausting retries, thepayment MUST be rolled back to its last confirmed state and any reservednonce or optimistic balance change MUST be released. No partial settlementmay persist.
INV-15: Submission Observability

Every submission attempt MUST emit structured logs and metrics coveringoutcome (success/failure), latency, retry count, and a stable error code.Metrics and logs MUST NOT expose secrets, private keys, or unnecessarypersonal data.
