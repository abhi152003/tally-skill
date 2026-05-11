# Company Profiles — Per-Client Voucher Behavior

Company profiles capture how a specific client company records vouchers in Tally. They are local markdown files under `company-profiles/` and act as the company-specific source of truth for voucher-entry rules.

Use profiles to reduce repeated CA questions while still respecting each client company's way of accounting.

## Core rule

Before working on any company, check whether a matching profile exists.

Match by:

1. Exact Tally company name.
2. Tally company GUID, if available.

If more than one profile matches, ask the CA to choose. Do not guess.

## Storage

Profiles live in:

```plain text
company-profiles/
```

Recommended filename:

```plain text
company-profiles/{normalized-company-name}-{short-guid}.md
```

Example:

```plain text
company-profiles/gokul-traders-4f7a91.md
```

## When a profile exists

1. Load the profile before preparing any voucher.
2. Match the request/invoice to a known voucher pattern.
3. Validate referenced masters for that pattern once per session:
   - Voucher type
   - Voucher class
   - Party ledger
   - Purchase/Sales ledger
   - GST ledgers
   - Stock items/UOM, if inventory mode is used
4. If validation passes, use the profile pattern to prepare the voucher.
5. For posting, ask for confirmation only once per company + voucher pattern + session. Avoid repeated confirmation for the same known pattern in the same session.

The profile is the source of intent. Tally is the source of existence. If the profile references a missing or renamed master, ask the CA how to map or fix it.

## When no profile exists

### Existing Tally company

If the company already has vouchers:

1. Fetch recent vouchers and masters.
2. Infer a profile draft.
3. Show the draft to the CA.
4. Save it only after CA approval.

Use recent data:

- Last 20 Sales vouchers, if available
- Last 20 Purchase vouchers, if available
- Last 20 Payment/Receipt/Journal vouchers where relevant
- Voucher type masters and classes
- Ledger masters referenced by sampled vouchers
- Stock items/UOM only if inventory vouchers are present

If a pattern cannot be inferred confidently, leave it blank and ask targeted questions when that pattern is first needed.

### New Tally company

If this is a newly created company:

1. Ask the CA targeted questions needed to create the profile.
2. Or, if CA says to follow another existing company, copy that profile structure.
3. When copying, change identity fields and mark rules/examples as inherited from the source company.

Do not silently create a profile for a new company without CA input.

GSTIN is optional at company creation time. TallyPrime's base Company Creation screen does not require GSTIN. If the CA provides GSTIN or wants GST configured now, capture it in the profile and validate it later against Tally's GST/Tax Unit setup. If the CA does not provide GSTIN, leave it blank and do not block company/profile creation.

## Conflict handling

If the CA asks for an entry that differs from the saved profile:

1. Explain the existing profile pattern briefly.
2. Ask whether to proceed with the different entry.
3. If CA confirms, post the voucher that way.
4. After posting, ask whether this should be added to the profile for future use.
5. If CA approves updating the profile, append a new exception pattern by default.

Only modify the default pattern if the CA explicitly says this is the new default.

## Profile structure

Use structured markdown with YAML frontmatter.

Required frontmatter:

```yaml
---
profile_version: 1
company_name: "COMPANY_NAME_AS_IN_TALLY"
company_guid: "TALLY_COMPANY_GUID_IF_AVAILABLE"
normalized_name: "normalized-company-name"
status: "approved"
created_at: "YYYY-MM-DD"
updated_at: "YYYY-MM-DD"
source:
  type: "inferred_from_tally" # inferred_from_tally | ca_interview | copied_profile
  copied_from: ""
validation:
  last_validated_session: ""
  notes: ""
---
```

Recommended sections:

````markdown
# Company Voucher Profile: COMPANY_NAME

## Identity
- Tally company name:
- Tally GUID:
- Business type:
- GSTIN: optional; fill only if CA provides it or GST setup is configured
- State:

## Session validation notes
- Referenced masters validated this session:
- Missing/renamed masters:

## Voucher patterns

### Sales — default
- Status: approved
- Mode: inventory_invoice | accounting_invoice | accounting_voucher
- Voucher type:
- Voucher class:
- Party ledger role:
- Sales ledger:
- GST ledgers:
- Inventory required:
- Quantity/rate required:
- Bill-wise allocation:
- Round off ledger:
- Confirmation scope:

#### Rules
- ...

#### Real examples
- Voucher no/date/party/amount:
- Ledger structure:
- Inventory structure:

#### XML skeleton
```xml
<!-- Minimal company-specific XML shape only. Use placeholders for dynamic values. -->
```

### Sales — exceptions
#### Exception: NAME
- Trigger:
- Difference from default:
- Approved by CA:
- Added on:

### Purchase — default
...

### Purchase — exceptions
...

### Payment — default
...

### Receipt — default
...

### Journal — default
...

## Changelog
- YYYY-MM-DD — Created profile from recent Tally vouchers; approved by CA.
- YYYY-MM-DD — Added Purchase exception for freight bills; approved by CA.
````

## Pattern fields

Each voucher pattern should capture:

- Posting mode:
  - `inventory_invoice`
  - `accounting_invoice`
  - `accounting_voucher`
- Voucher type name exactly as in Tally
- Voucher class name, if used
- Ledger names exactly as in Tally
- Whether inventory entries are required
- Whether quantity/rate are required
- GST ledger behavior
- Bill-wise allocation behavior
- Round-off behavior
- Minimal XML skeleton
- Real examples from the company, if available
- Exceptions approved by CA

## Approval rules

- Initial inferred profile: save only after CA approval.
- New company profile: create from CA answers or copied profile only.
- Conflict entry: post after CA confirms.
- Profile update after conflict: update only after CA approval.
- Default changes: require explicit CA statement that the default should change.
- Exception additions: append by default after CA approval.

## Session memory rules

Within the current session, remember:

- Company profile loaded
- Company + voucher pattern approvals
- Referenced masters already validated

Do not ask repeated confirmation for the same company + voucher pattern in the same session once CA has approved it.

## Source-of-truth hierarchy

1. Tally company data decides whether a master exists.
2. Company profile decides intended voucher behavior.
3. CA confirmation decides conflicts, missing mappings, and profile updates.
4. Global skill reference files provide fallback XML syntax and safety rules.
