---
name: Authorize and retrieve Green Button energy usage under NAESB ESPI
description: >-
  Use the NAESB REQ.21 ESPI (Green Button) model to obtain a retail customer's authorization at
  a utility Data Custodian and walk the Atom resource graph down to interval readings. NAESB
  authors this standard and hosts no endpoint — every call goes to the utility.
api: schemas/naesb-espi_v4.xsd
operations:
  - ApplicationInformation
  - Authorization
  - UsagePoint
  - MeterReading
  - ReadingType
  - IntervalBlock
  - IntervalReading
  - LocalTimeParameters
  - ElectricPowerUsageSummary
---

# Authorize and retrieve Green Button energy usage under NAESB ESPI

## The single most important fact

**NAESB operates no ESPI endpoint.** NAESB writes the REQ.21 Model Business Practices; each
utility Data Custodian implements them at its own host with its own authorization server. Any
plan that involves "calling the NAESB Green Button API" is wrong. Resolve the utility first,
then work against that utility's documented ESPI base URL.

Version 4.0.20231213 is current (`schemas/naesb-espi_v4.xsd`, `targetNamespace
http://naesb.org/espi`). Version 3.3 (`schemas/naesb-espi.xsd`) is still deployed widely —
check which generation the Data Custodian serves before validating payloads.

## What is open and what is not

The XML schemas in `schemas/` are released under Apache License 2.0 as a documented one-time
exception to the NAESB Copyright Policy, so you can validate and generate bindings from them
freely. The narrative Model Business Practices — including the scope-string grammar — are
copyright-protected and paywalled: $8,000/year membership, $2,000 per quadrant version, or
$250 per individual standard, with a free view-only three-business-day evaluation waiver for
non-members. Do not guess scope values from the schema; the schema declares a `scope` element
but publishes no vocabulary for it.

## Steps

1. **Register the third-party application.** The Data Custodian issues an
   `ApplicationInformation` resource. Read the OAuth endpoints out of it rather than
   hardcoding them — the whole point of the v4.0 design is that they vary per utility:
   `authorizationServerUri`, `authorizationServerAuthorizationEndpoint`,
   `authorizationServerTokenEndpoint`, `authorizationServerRegistrationEndpoint`. Also read
   `GrantType` and `TokenEndPointMethod` to learn which OAuth 2.0 flow and which client
   authentication method the utility expects, and `dataCustodianApplicationStatus` to confirm
   the registration is live.

2. **Send the retail customer to authorize.** Drive the OAuth 2.0 authorization code flow at
   `authorizationServerAuthorizationEndpoint`. Exchange the code at
   `authorizationServerTokenEndpoint`. If the utility supports dynamic client registration,
   `authorizationServerRegistrationEndpoint` is where that happens.

3. **Read the grant.** Fetch the `Authorization` resource. Check `status` and, critically,
   `authorizedPeriod` — the customer's grant is time-bounded, and reading outside it is a
   contract violation, not just an error. `scope` records what was actually granted; enforce
   it client-side rather than assuming your requested scope was honoured. The `error` element
   carries `OAuthError` values when the grant failed.

4. **Walk to the usage points.** From the authorization, follow the Atom `related` links to
   `UsagePoint` resources. Each carries `ServiceCategory` (which commodity),
   `ServiceStatus`, `amiBillingReady` and `connectionState`. Filter to the service category
   you actually need before pulling readings.

5. **Resolve the measurement semantics before the numbers.** For each `MeterReading`, fetch its
   `ReadingType` first. `ReadingType` carries `uom`, `powerOfTenMultiplier`, `commodity`,
   `flowDirection`, `accumulationBehaviour`, `dataQualifier`, `kind`, `measuringPeriod`,
   `phase` and `timeAttribute`. Interval values are meaningless without it — in particular,
   `powerOfTenMultiplier` must be applied and `flowDirection` distinguishes consumption from
   export.

6. **Pull the intervals.** `MeterReading` links to `IntervalBlock`, which contains
   `IntervalReading` elements, which may each carry `ReadingQuality` qualifiers. Treat any
   reading with a quality qualifier as suspect for billing-grade use.

7. **Get the clock right.** Fetch `LocalTimeParameters` for the usage point and apply
   `tzOffset`, `dstOffset`, `dstStartRule` and `dstEndRule` when converting interval
   timestamps. Interval data straddling a DST boundary is the classic source of silently wrong
   Green Button analytics.

8. **Prefer summaries when you only need totals.** `ElectricPowerUsageSummary`,
   `UsageSummary` and `ElectricPowerQualitySummary` give aggregated figures without walking
   every interval. For bulk subscription delivery the schema defines `BatchList` and
   `BatchItemInfo`.

## Customer-side resources

`schemas/naesb-customer_v4.xsd` (`targetNamespace http://naesb.org/espi/customer`) carries the
account and premises graph: `Customer`, `CustomerAccount`, `CustomerAgreement`,
`ServiceLocation`, `ServiceSupplier`, `Meter`, `EndDevice`, `Statement`, `PricingStructure`,
`DemandResponseProgram` and `DRProgramNomination`. Not every Data Custodian exposes these —
availability is a per-utility question.

## Certification

NAESB does **not** certify Green Button / ESPI implementations. That is done by the Green
Button Alliance under a separate collaborative arrangement with NAESB. NAESB's own
certification programme covers WEQ OASIS and WGQ standards only, and its published certified
products directory lists three certified gas products and zero certified WEQ OASIS products.
Do not treat a "NAESB certified" claim about a Green Button implementation at face value.

See `data-model/naesb-data-model.yml` for the full entity graph and
`authentication/naesb-authentication.yml` for the authorization model.
