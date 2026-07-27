# NAESB (naesb)

The North American Energy Standards Board (NAESB) is the non-profit, industry-consensus standards development organization formed in 1994 that writes the business practice standards for the North American wholesale and retail natural gas and electricity markets, organized into four quadrants — Wholesale Electric (WEQ), Retail Electric (REQ), Wholesale Gas (WGQ) and Retail Gas (RGQ). Headquartered in Houston, Texas, its home market is the United States (with Canadian and Mexican participation). NAESB sits upstream of every utility, ISO/RTO and energy-data platform in the value chain: it authors REQ.21 Energy Services Provider Interface (ESPI), the standard that is the basis of every Green Button implementation in North America, and it operates the NAESB Electric Industry Registry (EIR) that underpins electronic tagging across the wholesale electric market. Its API posture is deliberately split and must not be overstated. NAESB is not a data holder and no consumer-data mandate applies to it; the Green Button standard it publishes is adopted by US utilities purely voluntarily, with no federal obligation behind it. The specifications themselves are copyright-protected and paywalled — $8,000/year membership, $2,000 per quadrant version, or $250 per individual standard — with only a free, view-only, three-business-day evaluation waiver for non-members. The single genuinely open artifact is the set of ESPI XML schemas, released under Apache 2.0 as a documented one-time exception to the NAESB Copyright Policy and downloadable anonymously after a one-click terms-of-use acknowledgement. The one real API NAESB operates, the EIR webRegistry SOAP service administered by OATI, is closed: it requires a paid registry subscription and a digitally signed X.509 certificate issued by an NAESB-Authorized Certification Authority, and its endpoint could not even complete a TLS handshake anonymously. Open standard schemas, closed standards text, closed registry API, no consumer data and no open market data.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/naesb/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/naesb/refs/heads/main/apis.yml)

## Tags

- Energy
- United States
- Standards
- Utilities
- Electricity
- Gas
- Green Button
- Smart Metering
- Energy Markets
- Grid

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### NAESB REQ.21 Energy Services Provider Interface (ESPI)

The NAESB REQ.21 ESPI Model Business Practices define the data exchange protocol for transferring retail energy usage information from a utility (Data Custodian) to a Third Party with the Retail Customer's authorization. ESPI is the industry-consensus standard that is the basis of every Green Button implementation in North America and is the standard endorsed by the Green Button Alliance. NAESB publishes the standard; it does not operate an ESPI endpoint — each Data Custodian utility hosts its own, so there is no NAESB base URL. The Atom/XML payload schemas (NAESBespiSchema.xsd and retailcustomer.xsd, v3.3 and v4.0) are released under the Apache License 2.0 as a one-time exception to the NAESB Copyright Policy and are harvested here verbatim. The narrative Model Business Practices themselves are copyright-protected and must be purchased or obtained through NAESB membership. The v4.0 schema encodes an OAuth 2.0 authorization model directly — ApplicationInformation carries authorizationServerAuthorizationEndpoint, authorizationServerTokenEndpoint, authorizationServerRegistrationEndpoint, GrantType, TokenEndPointMethod, scope and OAuthError.

- **Human URL:** [https://www.naesb.org/ESPI_Standards.asp](https://www.naesb.org/ESPI_Standards.asp)

#### Tags

- Green Button
- ESPI
- Energy Usage
- Smart Metering
- OAuth
- Retail Electric

#### Properties

- [XML Schema](schemas/naesb-espi_v4.xsd) — ESPI v4.0.20231213, target namespace `http://naesb.org/espi`
- [XML Schema](schemas/naesb-customer_v4.xsd) — Retail Customer v4.0.20231213, target namespace `http://naesb.org/espi/customer`
- [XML Schema](schemas/naesb-espi.xsd) — ESPI v3.3.20200320
- [XML Schema](schemas/naesb-customer.xsd) — Retail Customer v3.3.20200130
- [Documentation](https://www.naesb.org/ESPI_Standards.asp)
- [Pricing](https://www.naesb.org//misc/naesb_matl_order_espi_standards.pdf)
- [Terms of Service](https://www.naesb.org/copyright.asp)

### NAESB Electric Industry Registry (EIR) webRegistry Web Services

The NAESB Electric Industry Registry is the central repository of registry information used by the North American wholesale electric industry for electronic tagging; it replaced the NERC TSIN in 2012 and is administered for NAESB by Open Access Technology International (OATI) as OATI webRegistry. It exposes a documented SOAP 1.1 web service with methods including DownloadEntity, DownloadBA, DownloadPSE and DownloadACA, described across the 121-page OATI webRegistry Technical Guide v6.1. The service is HTTPS-only, and per section 2.5 of that guide no request is processed unless it carries a digitally signed certificate associated with a registered webRegistry user; those certificates are issued by NAESB-Authorized Certification Authorities. Access also requires a paid registry subscription. An anonymous probe of the production endpoint on 2026-07-27 failed at the TLS layer (curl exit 60, "unable to get local issuer certificate") because the server certificate does not chain to a public root — the API cannot be reached at all without the registry PKI.

- **Human URL:** [https://www.naesb.org/weq/weq_eir.asp](https://www.naesb.org/weq/weq_eir.asp)
- **Base URL:** `https://www.naesbwry.oati.com/cgi-bin/webplus.dll?Script=/naesbwry/WREG-Web-Services-Main.wml`

#### Tags

- Registry
- Wholesale Electric
- SOAP
- E-Tagging
- Grid

#### Properties

- [Documentation](https://www.naesb.org/weq/weq_eir.asp)
- [Technical Guide](https://www.naesb.org/pdf4/eir_webregistry_technical_guide_v6.1_1018.pdf)
- [User Guide](https://www.naesb.org/pdf4/eir_webregistry_user_guide_v5.0_1023.pdf)
- [Getting Started](https://www.naesb.org/pdf4/eir_webregistry_quick_start_guide_v3.0_1023.pdf)
- [FAQ](https://www.naesb.org/pdf4/eir_user_interface_faq_0921.pdf)
- [Pricing](https://www.naesb.org/pdf4/naesb_eir_subscription_fee010715.docx)
- [Authentication](https://www.naesb.org/pdf4/ac_authorities_2023.pdf) — Independent Authorized Certification Authorities

## Common Properties

- [Website](https://www.naesb.org/)
- [About](https://www.naesb.org/aboutus.asp)
- [Contact Us](https://www.naesb.org/contactus.asp)
- [Documentation](https://www.naesb.org/ESPI_Standards.asp)
- [Tools](https://www.naesb.org/naesb_tools.asp)
- [Certification](https://www.naesb.org/materials/certification.asp)
- [Certified Products](https://www.naesb.org/pdf2/cert_products.pdf)
- [Certification Authorities](https://www.naesb.org/pdf4/ac_authorities_2023.pdf)
- [Pricing](https://www.naesb.org/pdf/ordrform.pdf)
- [Licensing](https://www.naesb.org/copyright.asp)
- [Terms of Service](https://www.naesb.org/pdf4/terms&conditions.pdf)
- [Privacy Policy](https://www.naesb.org/privacy.asp)
- [Membership](https://www.naesb.org/membership.asp)
- [Newsletter](https://www.naesb.org/bulletin_newsletter.asp)
- [Press Releases](https://www.naesb.org/news.asp)
- [White Papers](https://www.naesb.org/white_papers.asp)

## Mandate and Access Posture

| Question | Finding |
| --- | --- |
| Mandate regime | `green-button-voluntary` — NAESB authors Green Button; no US federal mandate compels adoption |
| Mandate status | `not-applicable` — NAESB is a standards body, not a data holder; it implements nothing |
| Data standard | NAESB REQ.21 ESPI v4.0 (Green Button); also WEQ/REQ/WGQ/RGQ business practice standards, WEQ-012 PKI, OASIS S&CP |
| Consumer data API | No — NAESB holds no customer data and operates no ESPI endpoint |
| Open market data | No — the EIR is subscription and certificate gated; nothing anonymous beyond schemas and PDF directories |
| Access gate | `accredited-only` — the EIR API needs a paid subscription plus an X.509 certificate from an NAESB-Authorized Certification Authority |
| Auth model | mTLS / signed X.509 client certificate for the EIR SOAP service; OAuth 2.0 required *of implementers* by the ESPI schema |
| Free artifacts | The four ESPI XML schemas only, under Apache License 2.0 — a documented one-time exception to the NAESB Copyright Policy |
| Paywall | $8,000/year membership · $2,000 per quadrant version · $250 per standard · free three-business-day view-only DRM evaluation |

Full probe log, HTTP statuses and verbatim source quotations are in [review.yml](review.yml).

## Maintainers

- Kin Lane — kin@apievangelist.com
