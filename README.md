# Allocation Readiness v0.1

A machine-readable readiness layer for determining whether an evidence package can proceed to allocation review.

It does **not** assign royalties, execute payments, determine ownership, establish origin, prove plagiarism, or make legal claims.

---

## Overview

Allocation Readiness v0.1 defines a minimal evidence-readiness object for structured review workflows.

It is designed to sit after comparison and lineage analysis, and before any downstream allocation or royalty process.

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

The central principle is:

Allocation Readiness is not allocation.
Readiness is not royalty assignment.
Review-readiness is not payment execution.

This specification only answers one question:

Is this evidence package ready to enter allocation review?

Purpose

Allocation Readiness v0.1 helps determine whether a package of evidence is sufficiently documented, reviewed, and policy-bounded to move into a later allocation review process.

It can be used to evaluate evidence packages that include:

comparison results
lineage relations
structure fingerprint references
provenance signals
timestamp evidence
dispute status
uncertainty records
human or multi-agent review notes

The goal is not to decide the final outcome.

The goal is to decide whether the evidence is ready for the next stage.

Non-Goals

This specification does not:

assign royalties
calculate payment shares
execute payments
determine authorship
determine origin
prove plagiarism
prove infringement
create legal claims
replace human review
replace governance review
replace dispute resolution

Allocation Readiness is a gateway, not a verdict.

Core Principle

Allocation workflows can be dangerous if weak evidence is converted directly into automatic decisions.

This specification prevents that by separating:

evidence package
  ≠
allocation decision
  ≠
payment execution
  ≠
legal judgment

In other words:

Evidence may become review input.
Review input must not become automatic enforcement.

Evidence Flow

Allocation Readiness v0.1 is designed to receive upstream evidence objects.

Comparison Result
  - How similar are the compared records?

Lineage Relation
  - What kind of structural relationship may exist?

Allocation Readiness
  - Is the evidence package ready for allocation review?

A valid Allocation Readiness object may allow an evidence package to proceed to review.

It must not automatically assign value, money, ownership, or legal responsibility.

Repository Structure
.
├── schemas/
│   └── allocation-readiness-v0.1.schema.json
│
├── examples/
│   └── allocation-readiness.sample.json
│
├── tests/
│   └── vectors/
│       ├── pass/
│       │   └── allocation-readiness.valid.json
│       │
│       └── fail/
│           ├── allocation-readiness.missing-decision.json
│           ├── allocation-readiness.auto-allocation-violation.json
│           └── allocation-readiness.missing-prohibited-use.json
│
├── .github/
│   └── workflows/
│       └── validate-specs.yml
│
├── LICENSE
└── README.md
Main Files
JSON Schema
schemas/allocation-readiness-v0.1.schema.json

Defines the machine-readable structure of an Allocation Readiness object.

The schema validates:

required namespaces
required fields
input references
evidence sufficiency status
readiness class
readiness score
review gates
uncertainty records
human review requirements
prohibited downstream uses
prevention of automatic allocation
Sample Object
examples/allocation-readiness.sample.json

Provides a valid example Allocation Readiness object.

The sample represents a conditionally ready evidence package that may proceed to allocation review, while still requiring human or multi-wing review.

Pass Vector
tests/vectors/pass/allocation-readiness.valid.json

A valid Allocation Readiness object that should pass schema validation.

Fail Vectors
tests/vectors/fail/allocation-readiness.missing-decision.json
tests/vectors/fail/allocation-readiness.auto-allocation-violation.json
tests/vectors/fail/allocation-readiness.missing-prohibited-use.json

These intentionally invalid objects test important safety boundaries.

They verify that:

recommended_next_step is required
automatic allocation cannot be allowed
automatic payment execution must be explicitly prohibited
Schema Usage

The schema uses JSON Schema Draft 2020-12.

schemas/allocation-readiness-v0.1.schema.json

It requires the following top-level namespaces:

meta
inputs
evidence
readiness
gates
uncertainty
review
policy
Required Namespaces
meta

Identifies the Allocation Readiness record.

Includes:

allocation_readiness_id
spec_version
created_at
producer
inputs

References upstream evidence objects.

May include:

lineage relation references
comparison result references
structure fingerprint references
evidence

Describes the sufficiency of the evidence package.

Includes:

evidence_status
sufficiency_score
evidence_summary
supporting evidence
missing evidence
conflicting evidence
readiness

Describes whether the evidence package may proceed to allocation review.

Includes:

readiness_class
readiness_score
allocation_review_allowed
automatic_allocation_allowed
recommended_next_step

Important rule:

automatic_allocation_allowed must always be false.
gates

Represents review gates that must be checked before downstream review.

Current gates:

provenance gate
lineage gate
uncertainty gate
dispute gate
policy gate
uncertainty

Preserves uncertainty from upstream evidence.

Includes:

confidence
ambiguity level
inherited uncertainty
unresolved issues
review

Describes whether human or multi-agent review is required.

Includes:

human_review_required
review_reason
recommended_review_type
review notes
policy

Defines allowed and prohibited downstream uses.

The policy section must explicitly prohibit:

automatic authorship claims
automatic origin claims
automatic plagiarism claims
automatic royalty assignment
automatic payment execution
automatic legal claims
irreversible enforcement
Local Validation

Install dependencies:

python -m pip install --upgrade pip
pip install jsonschema

Run validation locally:

python - <<'PY'
import json
import sys
from pathlib import Path

from jsonschema import Draft202012Validator, FormatChecker

print("=== Validating Allocation Readiness v0.1 ===")

schema_path = Path("schemas/allocation-readiness-v0.1.schema.json")
sample_path = Path("examples/allocation-readiness.sample.json")

pass_vectors = [
    Path("tests/vectors/pass/allocation-readiness.valid.json"),
]

fail_vectors = [
    Path("tests/vectors/fail/allocation-readiness.missing-decision.json"),
    Path("tests/vectors/fail/allocation-readiness.auto-allocation-violation.json"),
    Path("tests/vectors/fail/allocation-readiness.missing-prohibited-use.json"),
]

required_paths = [
    schema_path,
    sample_path,
    *pass_vectors,
    *fail_vectors,
]

missing_paths = [path for path in required_paths if not path.exists()]

if missing_paths:
    print("ERROR: Missing required files:")
    for path in missing_paths:
        print(f"  - {path}")
    sys.exit(1)

def load_json(path: Path):
    try:
        with path.open("r", encoding="utf-8") as f:
            return json.load(f)
    except json.JSONDecodeError as e:
        print(f"ERROR: Invalid JSON in {path}: {e}")
        raise

schema = load_json(schema_path)

validator = Draft202012Validator(
    schema,
    format_checker=FormatChecker()
)

def validate_should_pass(path: Path) -> bool:
    instance = load_json(path)
    errors = sorted(
        validator.iter_errors(instance),
        key=lambda e: list(e.path)
    )

    if errors:
        print(f"ERROR: Expected PASS but validation failed: {path}")
        for error in errors:
            location = ".".join(str(p) for p in error.path) or "<root>"
            print(f"  - {location}: {error.message}")
        return False

    print(f"OK: {path} is valid.")
    return True

def validate_should_fail(path: Path) -> bool:
    instance = load_json(path)
    errors = sorted(
        validator.iter_errors(instance),
        key=lambda e: list(e.path)
    )

    if not errors:
        print(f"ERROR: Expected FAIL but validation passed: {path}")
        return False

    print(f"OK: {path} failed as expected.")
    for error in errors[:5]:
        location = ".".join(str(p) for p in error.path) or "<root>"
        print(f"  - {location}: {error.message}")

    return True

all_ok = True

print("")
print("=== Validating sample ===")
all_ok = validate_should_pass(sample_path) and all_ok

print("")
print("=== Validating pass vectors ===")
for path in pass_vectors:
    all_ok = validate_should_pass(path) and all_ok

print("")
print("=== Validating fail vectors ===")
for path in fail_vectors:
    all_ok = validate_should_fail(path) and all_ok

print("")

if all_ok:
    print("Allocation Readiness validation succeeded.")
    sys.exit(0)

print("Allocation Readiness validation failed.")
sys.exit(1)
PY
GitHub Actions Validation

This repository includes a GitHub Actions workflow:

.github/workflows/validate-specs.yml

The workflow validates:

schemas/allocation-readiness-v0.1.schema.json
examples/allocation-readiness.sample.json
tests/vectors/pass/allocation-readiness.valid.json
tests/vectors/fail/allocation-readiness.missing-decision.json
tests/vectors/fail/allocation-readiness.auto-allocation-violation.json
tests/vectors/fail/allocation-readiness.missing-prohibited-use.json

Expected behavior:

the sample must pass
the pass vector must pass
all fail vectors must fail
Validation Rules

The schema validates structural safety rules such as:

required namespaces must be present
input references must be explicit
evidence sufficiency must be classified
readiness class must be declared
readiness score must be between 0 and 1
allocation review permission must be explicit
automatic allocation must be impossible
high ambiguity requires human review
insufficient evidence cannot proceed as ready
dispute failure must block or redirect review
automatic payment execution must be prohibited
Important Safety Rule

The most important rule in this specification is:

automatic_allocation_allowed: false

This prevents an Allocation Readiness object from becoming an automatic payment or royalty execution mechanism.

Allocation Readiness can say:

This evidence may proceed to review.

It cannot say:

Pay this person automatically.

This boundary is essential.

Policy Boundary

Allowed downstream uses include:

allocation review input
dispute input
audit input
human review input
multi-wing review input
governance review input
research input

Prohibited downstream uses include:

automatic authorship claims
automatic origin claims
automatic plagiarism claims
automatic royalty assignment
automatic payment execution
automatic legal claims
irreversible enforcement
Relationship to Upstream Specs

Allocation Readiness v0.1 is designed to work after upstream evidence layers.

Structure Fingerprint

Provides the structured fingerprint of a work, text, concept, specification, or protocol.

Comparison Result

Compares two Structure Fingerprint records and records similarity, uncertainty, and review requirements.

Lineage Relation

Expresses an evidence-backed lineage hypothesis between compared records.

Allocation Readiness

Determines whether the evidence package is ready to enter allocation review.

Relationship to Royalty OS

Allocation Readiness v0.1 can serve as an input layer for a downstream Royalty OS or allocation review system.

However, it does not itself perform allocation.

Allocation Readiness
  ↓
Allocation Review
  ↓
Royalty / Value Distribution Logic
  ↓
Payment or Non-Payment Outcome

Only later governance or allocation layers may decide whether value distribution should occur.

Example Readiness Classes

The schema supports the following readiness classes:

ready_for_allocation_review
conditionally_ready_for_allocation_review
not_ready
blocked_by_dispute
blocked_by_uncertainty
blocked_by_policy
insufficient_evidence

These classes describe review readiness only.

They do not describe entitlement.

Example Recommended Next Steps

The schema supports the following next steps:

proceed_to_allocation_review
request_human_review
request_multi_wing_review
request_more_evidence
send_to_dispute_registry
block_allocation_process
archive_for_audit_only

This keeps the object procedural rather than judgmental.

Current Status
Allocation Readiness v0.1:
  JSON Schema: implemented
  sample JSON: implemented
  pass vector: implemented
  fail vectors: implemented
  GitHub Actions validation: implemented
  CI status: passing
Roadmap

Recommended next steps:

Add a one-page overview document
Add relationship documentation with Comparison Result and Lineage Relation
Add a semantic validation script
Add additional fail vectors for dispute and uncertainty gates
Add diagrams for the evidence-to-review flow
Add release tag v0.1.0

Possible future files:

docs/one-page-overview.md
docs/relationship-to-comparison-result.md
docs/relationship-to-lineage-relation.md
docs/relationship-to-royalty-os.md
tools/validate_semantics.py
Non-Interference Statement

This specification is intentionally conservative.

It does not interfere with legal systems, authorship claims, platform policies, royalty contracts, or payment infrastructure.

It only provides a structured evidence-readiness object.

The final decision must remain with appropriate human, institutional, contractual, legal, or governance processes.

License

This repository is released under the license specified in LICENSE.

Current repository license:

MIT License
Citation

Suggested citation label:

Allocation Readiness v0.1

Suggested citation text:

Allocation Readiness v0.1:
Allocation Readiness v0.1:
A machine-readable readiness layer for determining whether evidence packages can proceed to allocation review without assigning royalties or executing payments.
