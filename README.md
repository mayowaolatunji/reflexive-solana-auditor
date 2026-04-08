# **Reflexive Solana Auditor (RSA)**

Reflexive Solana Auditor (RSA) is a security analysis system designed to **eliminate false positives by validating its own findings before reporting them**.

Unlike traditional tools that stop at detection, RSA:

* Generates hypotheses
* Attempts to disprove them
* Constructs exploits
* Filters out-of-scope issues
* Outputs only **confirmed, actionable vulnerabilities**

---

## Skill Definition

```json
{
  "name": "reflexive-solana-auditor",
  "description": "A self-validating Solana/Rust security auditor that generates, challenges, and proves findings before reporting them. Only confirmed, in-scope vulnerabilities with working POCs are surfaced and persisted.",
  "inputs": {
    "codebase_path": "string",
    "out_of_scope": "array (optional)"
  },
  "outputs": {
    "findings": "array",
    "report_file": "validFindings.md"
  }
}
```

---

## Core Behavior (System Prompt)

```text
You are a Solana smart contract security auditor operating under a strict reflexive validation model.

Your responsibility is NOT to list possible issues, but to prove real vulnerabilities.

You MUST follow this pipeline:

-----------------------------------
STEP 1 — Detection
-----------------------------------
Identify potential vulnerabilities using pattern recognition and heuristics.
Be aggressive in generating hypotheses.

-----------------------------------
STEP 2 — Self-Interrogation
-----------------------------------
For each hypothesis:
- Reconstruct all account constraints (signer, writable, ownership, seeds).
- Trace full instruction execution flow.
- Analyze CPI calls and authority propagation.
- Identify implicit invariants enforced by the program.

-----------------------------------
STEP 3 — Invalidation Attempt
-----------------------------------
Attempt to disprove the finding:
- Look for hidden guards or constraints.
- Identify why the vulnerability might NOT be exploitable.
- If a blocking condition exists, discard the finding.

-----------------------------------
STEP 4 — Proof of Concept (MANDATORY)
-----------------------------------
Construct a realistic exploit:
- Define attacker-controlled inputs
- Specify accounts and transaction structure
- Execute the vulnerable instruction path
- Demonstrate a successful state change

If you cannot produce a working exploit path, DISCARD the finding.

-----------------------------------
STEP 5 — OUT-OF-SCOPE VALIDATION (MANDATORY)
-----------------------------------
Check whether the affected code is within scope.

If the finding depends on any file, module, or logic marked as out-of-scope:
- DISCARD the finding completely

-----------------------------------
STEP 6 — VERDICT
-----------------------------------
Classify findings as:
- CONFIRMED → exploit works and is in-scope
- FALSE_POSITIVE → invalidated during reasoning
- OOS → out of scope (discard)
- INCONCLUSIVE → insufficient evidence (do not report)

-----------------------------------
STEP 7 — REPORTING
-----------------------------------
ONLY for CONFIRMED findings:

1. Output the finding
2. Append it to `validFindings.md` as a clean audit report

-----------------------------------
OUTPUT REQUIREMENTS
-----------------------------------
Each finding MUST include:
- Title
- Location (file + function)
- Summary
- Invariant violated
- Step-by-step POC
- Impact
- Confidence score

-----------------------------------
CRITICAL RULES
-----------------------------------
- Never output unproven vulnerabilities
- Never output findings without a working POC
- Never output findings that are out-of-scope
- Precision is more important than coverage
```

---


---

## 2. Key Capabilities

### 2.1 Reflexive Validation

Every finding is treated as untrusted until it survives:

* constraint reconstruction
* execution reasoning
* adversarial analysis

---

### 2.2 Mandatory Exploit Construction

RSA does not report theoretical issues.

A finding is only valid if:

* a realistic exploit path exists
* the exploit produces a measurable state change

---

### 2.3 Out-of-Scope Enforcement

RSA ensures audit correctness by:

* filtering findings tied to excluded files or modules
* preventing invalid or irrelevant reports

---

### 2.4 Persistent Audit Reporting

All validated findings are written to:

```
validFindings.md
```

This file serves as:

* a clean audit artifact
* a ready-to-submit report
* a source of verified vulnerabilities only

---

## 3. Processing Pipeline

```
Codebase
   ↓
Detection Engine
   ↓
Self-Interrogation
   ↓
Exploit Construction
   ↓
Out-of-Scope Filter
   ↓
Confirmed Findings Only
   ↓
validFindings.md
```

---

## 4. Proof of Concept (POC) Standard

Each confirmed finding must include a structured POC:

### Required Sections

1. **Setup**

   * attacker-controlled inputs
   * account configuration

2. **Execution**

   * instruction call
   * CPI interactions (if applicable)

3. **Result**

   * observable state change
   * unauthorized behavior

4. **Explanation**

   * why constraints failed
   * how the exploit bypasses protections

---

## 5. `validFindings.md` Specification

Each finding is appended in the following format:

```markdown
## [SEVERITY] Finding Title

**Location:**
- path/to/file.rs::function_name

---

### Summary
Brief description of the vulnerability.

---

### Invariant Violated
Define the security property that is broken.

---

### Proof of Concept (POC)

#### 1. Setup
Describe attacker preparation and inputs.

#### 2. Execution
Describe how the exploit is triggered.

#### 3. Result
Show the resulting state change.

#### 4. Why It Works
Explain the root cause.

---

### Impact
Explain the consequence (e.g., fund loss, privilege escalation).

---

### Confidence
0.00 – 1.00
```

---

## 6. Out-of-Scope Configuration

### Example Input

```json
{
  "out_of_scope": [
    "tests/",
    "mocks/",
    "examples/",
    "lib/external/",
    "programs/legacy/"
  ]
}
```

---

### Enforcement Rules

* Any finding involving out-of-scope code is discarded
* Partial findings are not allowed
* Scope violations override exploit validity

---

## 7. Output Guarantees

RSA guarantees that every reported finding is:

* ✔ Exploitable
* ✔ Verified through reasoning
* ✔ Backed by a working POC
* ✔ Within defined audit scope

---

## 8. Usage Example

### Invocation

```json
{
  "tool": "reflexive-solana-auditor",
  "input": {
    "codebase_path": "./programs/vault",
    "out_of_scope": ["tests/", "mocks/"]
  }
}
```

---

### Result

* Structured findings returned
* `validFindings.md` created/updated with confirmed vulnerabilities

---

## 9. Design Outcome

RSA changes the audit workflow from:

> reviewing everything flagged

to:

> reviewing only what has already been proven

---

## 10. Summary

Reflexive Solana Auditor is built on a single principle:

> A vulnerability is not real until it survives an attempt to disprove it.

By combining:

* self-interrogation
* exploit validation
* strict scope enforcement

RSA produces **fewer findings — but only the ones that matter**.

---

