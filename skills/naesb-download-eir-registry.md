---
name: Download a NAESB EIR registry publication
description: >-
  Pull a point-in-time snapshot of the NAESB Electric Industry Registry — market participants,
  points, paths and adjacencies — from the OATI-administered webRegistry SOAP service, pinned
  to a specific registry publication version.
api: wsdl/naesb-eir-webregistry.wsdl
operations:
  - DownloadRegistryVersion
  - DownloadEntity
  - DownloadBA
  - DownloadPSE
  - DownloadTSP
  - DownloadPORPOD
  - DownloadSourceSink
  - DownloadTransmissionPath
  - DownloadBAAdjacency
---

# Download a NAESB EIR registry publication

## Before you start

You cannot complete this flow without credentials. The service refuses every operation
anonymously with `ReturnCode 1 / FAILURE` and the message
`Please present a valid certificate that is associated with a NAESB EIR user`. You need:

1. A paid NAESB EIR registration and current annual subscription ($275 as of 2025-10-01).
2. An X.509 client certificate issued by a NAESB-Authorized Certification Authority and
   associated with a registered EIR user. See `authentication/naesb-authentication.yml`.
3. TLS 1.2 or better. Note that the server certificate is issued by OATI's private
   `webCARES Server Issuing CA 2025` and does **not** chain to a public root — you must trust
   the OATI CA explicitly. A default trust store will fail with "unable to get local issuer
   certificate". Do not work around this by disabling verification in production.

Endpoint:
`https://www.naesbwry.oati.com/cgi-bin/webplus.dll?script=/NAESBWRY/WREG-Web-Services-Main.wml`

## Request shape

Every one of the 30 operations is read-only and takes the same two optional inputs,
`EffectiveDate` and `Version`. Two namespaces are in play and getting them wrong is the most
common failure:

- The **operation element** belongs to `http://www.oati.net/namespace`.
- The **types** (`OutputStruct`, `ErrorStruct`, every `*Struct`) belong to
  `http://www.oati.net/namespace/Registry/WREG_WebServices`.

Set `SOAPAction: http://www.oati.net/namespace/{OperationName}` and
`Content-Type: text/xml; charset=utf-8`. If you emit the operation element in the types
namespace you get HTTP 500 and a `soap:Fault` reading
`Document contains an element <X> that is not defined in the schema.`

## Steps

1. **Pin the publication.** Call `DownloadRegistryVersion` first. It returns
   `RegistryVersionStruct` rows with `VersionID`, `VersionCode`, `EffectiveDate`,
   `PublicationMethodCode` and `PublicationStatus`. Select the row with
   `PublicationStatus = ACTIVE`, or a `FUTURE` row if you are staging against an upcoming
   publication. Carry its `VersionCode` as the `Version` input on every subsequent call so the
   whole snapshot is internally consistent — do not mix versions across operations.

2. **Pull the participants.** Call `DownloadEntity` for the root entity records, then the
   role-specific operations you need: `DownloadBA` (Balancing Authorities), `DownloadPSE`
   (Purchasing-Selling Entities), `DownloadTSP` (Transmission Service Providers). Join role
   records back to entities on `TaggingEntityID`, not on the local `ID`.

3. **Pull the points.** Call `DownloadPORPOD` for points of receipt and delivery, and
   `DownloadSourceSink` for generation sources and load sinks. Both key on `TaggingPointID`,
   which is the join key used by paths and adjacencies.

4. **Pull the topology.** Call `DownloadTransmissionPath` for registered POR-to-POD paths, then
   `DownloadBAAdjacency` for the BA-to-BA edges. The adjacency structs use directional
   `From*` / `To*` field pairs — see `data-model/naesb-data-model.yml` for the full edge set,
   including `DownloadPODPORAdjacency`, `DownloadSinkPODAdjacency` and
   `DownloadSourcePORAdjacency`.

5. **Respect effective dating.** Nearly every struct carries `StartEffectiveDate` and
   `StopEffectiveDate`, and participant roles also carry `ExpirationDate`. Filter locally to
   your date of interest, or set the `EffectiveDate` input so the server does it. A record
   present in the publication is not necessarily in effect on your date.

## Reading the response

There is no HTTP-status-based error signalling. An application failure still returns HTTP 200.
Always check the envelope before touching the payload:

- `OutputStruct.ReturnCode` and `ReturnCodeDesc` carry the outcome. `1` / `FAILURE` means it
  did not work.
- On failure, `OutputStruct.Error` holds one or more `ErrorStruct` with `ErrorCode`,
  `ErrorCodeDesc` and an optional `ParentElementIndex` pointing at the offending input row.
- On success, `OutputStruct.Success` holds the requested `*Struct` rows.

See `errors/naesb-error-codes.yml`.

## Things that do not exist — do not look for them

There is no pagination, so a `Download*` call returns the entire result set for the requested
date and version; size your client accordingly. There are no per-field query filters beyond
`EffectiveDate` and `Version`. There is no idempotency key and none is needed — every
operation is a safe read. There is no rate-limit header, no request-id header, no webhook and
no status page. Operational support is a phone line: the OATI Help Desk at 763-201-2020.

## Staying current

Registry changes are announced by dated notice on
`https://www.naesb.org/weq/weq_eir.asp`, not by an API. Recent changes have altered CSV import
behaviour, removed the legacy WECC reliability area, and corrected Entity State/Country
fields. Track `changelog/naesb-changelog.yml` and re-check `DownloadRegistryVersion` for
`PENDING` and `FUTURE` publications before a deployment.
