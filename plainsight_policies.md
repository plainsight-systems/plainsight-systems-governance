---
name: Plainsight Systems parent-organization policies
description: Parent-organization policies that apply across Plainsight Systems LLC operating brands.
type: project
---

# Plainsight Systems Policies

These policies apply across Plainsight Systems LLC products unless a product has
a stricter rule. If a product-specific decision conflicts with these policies,
escalate and revisit the policy rather than silently deviating.

## Brand and Entity Boundaries

Plainsight Systems LLC is the parent and contracting entity. PlainSight Lab,
Appario, and CustodyZero are operating brands with distinct audiences,
repositories, and product promises.

Concrete rules:

- Do not blur the parent entity with operating brands in legal, billing, or public-contact language.
- Product repos should name the operating brand first and the parent entity where legal or governance context matters.
- Cross-brand decisions belong in this governance repo or the relevant ops/legal source, not only inside one product repo.

## Data Handling Honesty

Products must describe data flows as they actually exist, not as marketing
intent. Any claim about privacy, custody, analytics, AI use, or retention must
be backed by architecture and reviewable implementation.

Concrete rules:

- No hidden telemetry paths.
- No silent cloud fallback for local or installed products.
- No customer or merchant data should be introduced into committed fixtures unless it is synthetic or explicitly approved for that repository.
- Sensitive operational facts belong in `plainsight-systems-ops` pointers or local ignored files, not committed product docs.

## Security Updates

Security-class fixes should remain available to supported users without being
used as a payment lever. Product-specific licensing may differ, but withholding
security fixes to force upgrades conflicts with the parent posture.

Any end-of-life proposal for a shipped product must include:

- affected versions,
- transition window,
- user-facing notice plan,
- upgrade or migration path,
- explicit security-support cutoff.

## No Facades

Plainsight products do not pretend an unfinished path works. Stubs,
placeholder success responses, demo-only shortcuts, and "temporary" silent
fallbacks must fail loudly or be clearly kept outside the product path.

This rule applies to docs too: if a product direction is aspirational, label it
as planned or open rather than documenting it as shipped.
