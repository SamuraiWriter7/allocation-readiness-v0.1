# Allocation Readiness v0.1 — One-Page Overview

Allocation Readiness v0.1 is a machine-readable readiness layer for determining whether an evidence package can proceed to allocation review.

It does **not** assign royalties, execute payments, determine authorship, establish origin, prove plagiarism, or make legal claims.

---

## 1. Core Question

This specification answers one question:

> Is this evidence package ready to enter allocation review?

It does not answer:

> Who should be paid?  
> How much should they receive?  
> Who is the legal owner?  
> Who is the original author?

Allocation Readiness is a gateway into review, not a final decision engine.

---

## 2. Evidence Flow

Allocation Readiness v0.1 sits after comparison and lineage analysis.

```text
Structure Fingerprint
  ↓
Comparison Result
  ↓
Lineage Relation
  ↓
Allocation Readiness
  ↓
Allocation / Royalty Review Layer

Each layer has a different role.

Comparison Result
  → records similarity and uncertainty

Lineage Relation
  → records a possible structural relationship

Allocation Readiness
  → records whether the evidence is ready for allocation review
3. What It Can Do

Allocation Readiness v0.1 can describe:

whether evidence is sufficient for review
whether uncertainty remains
whether disputes are known
whether provenance is strong or weak
whether human review is required
whether the package should proceed, pause, or be blocked
whether downstream use must be limited

It can produce outputs such as:

ready_for_allocation_review
conditionally_ready_for_allocation_review
not_ready
blocked_by_dispute
blocked_by_uncertainty
blocked_by_policy
insufficient_evidence

These are review-readiness states, not entitlement decisions.

4. What It Cannot Do

Allocation Readiness v0.1 must not be used for:

automatic authorship claims
automatic origin claims
automatic plagiarism claims
automatic royalty assignment
automatic payment execution
automatic legal claims
irreversible enforcement

The most important safety rule is:

automatic_allocation_allowed: false

This ensures that readiness never becomes automatic payment or enforcement.

5. Main Object Structure

An Allocation Readiness object contains eight top-level namespaces:

meta
inputs
evidence
readiness
gates
uncertainty
review
policy
meta

Identifies the readiness record.

inputs

References upstream evidence objects, such as comparison results, lineage relations, and structure fingerprints.

evidence

Describes whether the evidence package is sufficient, partial, conflicting, disputed, or unavailable.

readiness

States whether allocation review is allowed and what the next step should be.

gates

Checks key review gates:

provenance gate
lineage gate
uncertainty gate
dispute gate
policy gate
uncertainty

Preserves ambiguity and inherited uncertainty from upstream evidence.

review

Describes whether human, multi-wing, governance, legal, or allocation review is needed.

policy

Defines allowed and prohibited downstream uses.

6. Example Decision Pattern

A typical valid object may say:

The evidence package is conditionally ready for allocation review.
Human or multi-wing review is required.
Automatic allocation is not allowed.
Payment execution is prohibited.

In JSON terms, the key pattern is:

{
  "readiness": {
    "readiness_class": "conditionally_ready_for_allocation_review",
    "allocation_review_allowed": true,
    "automatic_allocation_allowed": false,
    "recommended_next_step": "request_multi_wing_review"
  }
}

This means:

The package may move forward, but only into review.

It does not mean:

Allocate royalties automatically.

7. Validation

This repository validates:

schemas/allocation-readiness-v0.1.schema.json
examples/allocation-readiness.sample.json
tests/vectors/pass/allocation-readiness.valid.json
tests/vectors/fail/allocation-readiness.missing-decision.json
tests/vectors/fail/allocation-readiness.auto-allocation-violation.json
tests/vectors/fail/allocation-readiness.missing-prohibited-use.json

Expected behavior:

valid sample passes
valid pass vector passes
invalid fail vectors fail

The fail vectors confirm that:

a next step is required
automatic allocation cannot be enabled
automatic payment execution must be prohibited
8. Relationship to Royalty OS

Allocation Readiness v0.1 may become an input layer for downstream Royalty OS or value-distribution systems.

However, it does not perform allocation.

Allocation Readiness
  ↓
Allocation Review
  ↓
Royalty / Value Distribution Logic
  ↓
Payment or Non-Payment Outcome

Only later governance, contractual, institutional, or review layers may decide whether value distribution should occur.

9. Design Philosophy

The design follows three principles:

Readiness is not entitlement.
Review input is not payment execution.
Evidence objects are not verdicts.

This keeps the system safe, auditable, and reviewable.

Allocation Readiness v0.1 exists to prevent weak or uncertain evidence from becoming automatic enforcement.

It is a checkpoint, not a judge.

10. Status
Allocation Readiness v0.1:
  JSON Schema: implemented
  sample JSON: implemented
  pass vector: implemented
  fail vectors: implemented
  GitHub Actions validation: implemented
  CI status: passing

作成後のコミットはこれです。

```bash
mkdir -p docs

git add docs/one-page-overview.md
git commit -m "Add one-page overview for Allocation Readiness v0.1"
git push
