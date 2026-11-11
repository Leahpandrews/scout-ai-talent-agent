# Scout Acceptance Test Cases

This file documents representative acceptance tests for the public, sanitized Scout case study.

> **Privacy note:** All names, organizations, email addresses, statuses, and records in this repository are synthetic. These tests describe generalized workflow behavior and do not expose production data, employer information, client information, or proprietary credentials.

## Testing Goals

Scout is designed as a human-in-the-loop AI workflow. Testing therefore focuses not only on whether the model produces useful output, but also on whether the system behaves safely and predictably when data is incomplete, conflicting, duplicated, restricted, or unsuitable for automated action.

The core acceptance criteria are:

- preserve source records and distinguish verified evidence from inference;
- never treat missing information as automatic evidence of fit or failure;
- separate fit evaluation from evidence confidence;
- route ambiguous or consequential cases to human review;
- respect opt-out and do-not-contact status before outreach;
- prevent cross-candidate data leakage;
- prevent duplicate draft creation;
- fail individual records without stopping a valid batch;
- create drafts only when action prerequisites are satisfied;
- never auto-send outreach as part of the default workflow.

---

## Test Matrix

| ID | Scenario | Synthetic Setup | Expected Behavior | Pass Criteria |
|---|---|---|---|---|
| T01 | Standard high-fit record | Complete candidate record with consistent source and CRM data | Candidate is scored, assigned evidence confidence, segmented, and marked outreach-ready | Scores reconcile; rationale uses supported evidence; status is `READY_TO_DRAFT` |
| T02 | Missing soft-fit information | Candidate meets hard requirements but one optional fit field is blank | Missing value remains unknown and receives no points | Candidate is not excluded solely because the field is missing |
| T03 | Unverified hard requirement | A project-defined hard requirement cannot be confirmed | Record is not automatically failed | Candidate is routed to `Manual Review` when the uncertainty is material |
| T04 | CRM do-not-contact override | Source data indicates strong fit; CRM contains active suppression status | Suppression overrides fit and stops outreach workflow | Record is `Excluded` / `DO_NOT_CONTACT`; no draft is created |
| T05 | Ambiguous CRM identity match | Multiple internal records match the same name without a decisive identifier | System does not guess which record is correct | Record is tagged `CRM_AMBIGUOUS_MATCH` and routed to `Manual Review` |
| T06 | No CRM match | Candidate has valid sourced data but no internal CRM record | Workflow continues using sourced data | `CRM_NO_MATCH` is logged and is not treated as a negative fit signal |
| T07 | Missing usable email | Candidate is otherwise eligible but has no valid email | Candidate may remain ranked but is not contact-ready | Fit is preserved; contact status prevents draft creation |
| T08 | Template not found | Candidate is assigned a template label that does not exist | Only that record fails | Error is `TEMPLATE_NOT_FOUND`; remaining valid records continue |
| T09 | Duplicate template label | Two templates use the same label | System does not choose one arbitrarily | Record fails with `DUPLICATE_TEMPLATE_LABEL` |
| T10 | Unresolved placeholder | Required template placeholder lacks verified source data | System does not invent a value | Record fails with `UNRESOLVED_PLACEHOLDER` |
| T11 | Cross-candidate leakage check | Two synthetic candidates have distinct names, roles, and personalization hooks | Each draft is validated against its own record | No field or personalization from Candidate A appears in Candidate B's draft |
| T12 | Duplicate draft prevention | Same campaign + candidate + template + version is processed twice | Second action is skipped unless regeneration is explicitly requested | Existing successful action key prevents a duplicate draft |
| T13 | Isolated record failure | One record in a batch has malformed data; others are valid | Batch continues after the failed record | Valid records complete; failed record is reported separately |
| T14 | Reply classification — interested | Synthetic reply clearly asks to schedule a conversation | Reply is classified as `INTERESTED` | Classification matches requested next step and follow-up is appropriate |
| T15 | Reply classification — not now | Synthetic reply requests contact at a later time | Reply is classified as `NOT NOW`, not `NOT INTERESTED` | Follow-up respects timing and does not create recurring contact without permission |
| T16 | Explicit opt-out reply | Synthetic reply says not to contact again | Promotional follow-up stops immediately | Suppression is triggered; no additional promotional draft is created |
| T17 | Draft-only safety | Valid outreach batch is approved for draft creation | System creates drafts but does not send | Drafts exist; nothing appears as automatically sent |
| T18 | Evidence confidence separation | Candidate has strong fit indicators but sparse or secondary evidence | Fit can remain high while confidence is lower | Output shows distinct `Total Fit Score` and `Evidence Confidence` values |

---

## Example Acceptance Run

A minimal pre-production test can use a fictional batch containing:

1. one high-fit candidate with a verified CRM match;
2. one candidate with an active do-not-contact flag;
3. one candidate assigned to a missing template;
4. one ambiguous CRM match;
5. two candidates using different templates and different personalization hooks.

### Expected Results

- the valid candidate receives one correctly matched draft;
- the opted-out candidate is skipped and reported;
- the missing-template record fails without stopping the batch;
- the ambiguous identity is held for manual review;
- candidate-specific information remains isolated to the correct draft;
- rerunning the same successful campaign does not create a duplicate draft;
- no outreach is automatically sent.

---

## Why These Tests Matter

The goal of Scout is not simply to generate plausible recruiting output. The system must also know when **not** to act.

The most important tests therefore exercise uncertainty, conflicting data, privacy controls, identity resolution, action boundaries, and recoverable failure states. This reflects a broader design principle used throughout the project:

> **Automate structured, reversible work while keeping ambiguous or consequential decisions visible to a human reviewer.**
