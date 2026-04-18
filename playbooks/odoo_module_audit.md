# Odoo Module Audit

Use this playbook to assess a third-party Odoo addon before adoption, during integration, or when reviewing a vendor update.

It is designed for external modules in general, with Odoo Apps Store addons as the most common example.

## What This Playbook Optimizes For

- Repeatable audits instead of impression-based reviews
- Findings that focus on operational and maintenance risk
- A recommendation that is useful for adoption decisions
- Output that can be shared with technical and non-technical stakeholders

## Default Posture

- Treat third-party addons as vendor code.
- Default to read-only review first.
- Prefer wrapping or extending the addon over editing vendor code directly.
- Treat license comments as engineering compatibility guidance, not legal advice.
- Qualify conclusions when the audit scope is narrow and runtime checks were not performed.

## When To Use It

- Evaluating whether to adopt a third-party addon
- Reviewing an addon that is already installed
- Reviewing a vendor update before deployment
- Deciding whether a defect should be solved by configuration, wrapper, vendor fix, fork, or rejection

## Declare The Audit Scope First

State the scope explicitly before giving conclusions:

- code review only
- code plus installability
- code plus tests

Do not imply runtime validation unless it was actually checked.

## Audit Workflow

1. Confirm the target module and audit goal:
   - pre-adoption evaluation
   - review of an installed addon
   - review of a vendor update
2. State the audit scope.
3. Read the module manifest, README, license files, security files, models, views, controllers, assets, tests, and translations as relevant.
4. Review the addon across the audit dimensions below.
5. Record findings first, ordered by severity, with file references where available.
6. Score the addon using the scorecard.
7. Finish with an explicit recommendation and a plain-language adoption summary.

## Audit Dimensions

### 1. Functional Fit

- Confirm the addon actually covers the requested business use case.
- Check overlap with standard Odoo or already-used modules.
- Note gaps between the sales promise and the actual implementation.

### 2. Packaging And Integration

- Review `__manifest__.py` dependencies, data files, assets, demo data, and external dependencies.
- Check whether security files, access rules, and views are complete.
- Note required repositories, Python packages, system packages, or deployment changes.
- Check whether the addon assumes Enterprise features or optional modules without declaring them clearly.

### 3. Code Quality And Maintainability

- Look for missing `super()`, monkey patches, copied core code, broad overrides, and brittle view inheritance.
- Review compute methods, constraints, onchange logic, and ORM usage for upgrade-safe patterns.
- Flag unsafe SQL, broad `sudo()`, recordset leakage to controllers, and portal/public exposure issues.
- Treat missing tests, weak assertions, and absent translations as maintenance risk, not just polish issues.

### 4. Upgrade And Migration Risk

- Check whether the code follows the conventions of the target Odoo version.
- Watch for private API usage, hard overrides of core methods, obsolete JS or OWL patterns, and fragile XML structures.
- Note whether uninstall, upgrade, or migration impact is unclear or likely unsafe.

### 5. License And Vendor Risk

- Check manifest license, headers, LICENSE file presence, and compatibility with the target environment.
- Call out mixed or unclear licensing, especially when the addon pulls incompatible dependencies.
- Note whether the module appears hard to maintain without vendor support.

### 6. Operational Risk

- Review cron jobs, queue usage, multi-company behavior, access rules, and performance-sensitive searches.
- Check import, mail, website, portal, and API flows when the addon touches those areas.
- Flag mass recompute risks, expensive loops, and missing indexes when searches are central to the design.

## Red Flags

Watch closely for these recurring adoption risks:

- missing `super()` where inheritance should be preserved
- monkey patches where a normal extension point exists
- broad or unjustified `sudo()`
- missing ACLs or weak record rules
- brittle xpath expressions or unnecessary `replace` operations
- missing meaningful Odoo tests
- unclear or incompatible licensing
- logic that breaks import, mail, portal, website, cron, or API flows
- unreviewed multi-company or performance side effects

## Review Tags

Use these tags exactly in findings:

- `[change]` mandatory fix required before approval
- `[question]` clarification or missing information required before approval
- `[comment]` non-blocking improvement or observation

Examples:

- `[change]` The module overrides a core method without `super()`, which makes the customization brittle across upgrades.
- `[change]` The controller exposes business data with broad `sudo()` and no clear access control boundary.
- `[question]` The module appears to depend on an Enterprise feature, but the dependency is not declared clearly enough to assess deploy risk.
- `[question]` It is unclear whether this behavior is intended for portal users, internal users, or both, which affects the security assessment.
- `[comment]` The inherited view could target the field directly instead of using a broader xpath, which would make the customization easier to maintain.
- `[comment]` The module would benefit from stronger Odoo tests around the main business flow even if the current implementation is acceptable.

## Scorecard

Score each category from 1 to 5 stars. Half-stars are acceptable.

Categories:

- functional fit
- code quality
- security
- upgrade safety
- test coverage
- documentation
- license/compliance
- maintainability

Weighted overall score:

- functional fit: 15%
- code quality: 20%
- security: 20%
- upgrade safety: 15%
- test coverage: 10%
- license/compliance: 10%
- maintainability: 5%
- documentation: 5%

Recommendation mapping:

- 4.5-5.0: approve
- 3.5-4.4: approve with minor fixes
- 2.5-3.4: conditional approval
- 1.5-2.4: major remediation required
- 0.0-1.4: reject

## Recommended Audit Output

Start with a short module header:

- module technical name
- vendor when known
- Odoo version reviewed
- audit date
- scope

Then use this structure:

1. Executive summary
2. Scorecard
3. Findings
4. Detailed review sections
5. Final recommendation

Group findings into:

- blocking
- major
- minor
- questions

End with both:

- a recommendation such as approve, conditional approval, or reject
- a short plain-language recommendation about adoption, containment, remediation, or rejection

## Suggested Detailed Review Headings

- functional fit
- manifest and packaging
- licensing
- dependencies and architecture
- Python / ORM
- views / XML / UI
- security
- standard Odoo flow compatibility
- tests
- translations and documentation
- upgrade and maintenance risk

## Preferred Remediation Order

1. Use the addon as-is when it is acceptable.
2. Extend it with a small wrapper or bridge module.
3. Ask the vendor for a fix when the defect belongs in the addon itself.
4. Fork or patch vendor code only when that tradeoff is explicitly accepted.

## Practical Notes

- A module with no meaningful Odoo tests should score poorly on test coverage.
- Security, code quality, and upgrade safety should dominate the recommendation when they are weak.
- Documentation polish should not hide structural risk.
- Keep the audit focused on adoption and maintenance risk, not cosmetic style comments.

## Reusable Audit Template

```md
Module: <technical_name>
Vendor: <vendor or unknown>
Odoo version: <version reviewed>
Audit date: <YYYY-MM-DD>
Scope: <code review only | code plus installability | code plus tests>

## Executive summary

<2-5 sentence summary of the overall recommendation and main risks>

## Scorecard

- Functional fit: <x/5>
- Code quality: <x/5>
- Security: <x/5>
- Upgrade safety: <x/5>
- Test coverage: <x/5>
- Documentation: <x/5>
- License/compliance: <x/5>
- Maintainability: <x/5>
- Weighted overall score: <x/5>
- Recommendation: <approve | approve with minor fixes | conditional approval | major remediation required | reject>

## Findings

### Blocking

- [change] <finding with file reference when available>

### Major

- [change] <finding with file reference when available>

### Minor

- [comment] <finding with file reference when available>

### Questions

- [question] <clarification needed before approval>

## Detailed review

### Functional fit

### Manifest and packaging

### Licensing

### Dependencies and architecture

### Python / ORM

### Views / XML / UI

### Security

### Standard Odoo flow compatibility

### Tests

### Translations and documentation

### Upgrade and maintenance risk

## Final recommendation

<short plain-language adoption recommendation>
```
