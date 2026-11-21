# AIP chart update index v1

## Discovery and signing

Fetch the [AIP update index](https://raw.githubusercontent.com/johnbenz13/fplnav-downloads/main/v1/aip-charts.json) over HTTPS. The response is a UTF-8 JSON object with three string fields.

| Field | Format |
|---|---|
| `keyId` | Identifier of a key trusted by the consumer. The first key is `aip-index-ed25519-v1`. |
| `payload` | Standard padded RFC 4648 base64 of the complete UTF-8 JSON payload. |
| `signature` | Standard padded RFC 4648 base64 of a 64-byte Ed25519 signature over the decoded payload bytes. |

Verify the signature over the decoded bytes before interpreting the payload. Use the trusted public key shipped with the consumer. The [published public key](../v1/aip-charts-key.json) is standard padded base64 of the 32 raw Ed25519 public key bytes. Pin that key through a trusted software release. Fetching a key alongside the envelope is insufficient to establish trust. The live envelope is a byte-exact interoperability fixture.

## Signed payload

| Field | Format and role |
|---|---|
| `schemaVersion` | Integer, currently `1`. |
| `dataset` | String, exactly `aip-charts`. |
| `generation` | Positive safe integer, increased for each publication or retraction. |
| `releases` | Array of currently eligible release descriptors. An empty array offers no update. |

A release descriptor contains:

| Field | Format and role |
|---|---|
| `sequence` | Positive safe integer. A higher sequence denotes a newer chart release. Independent of index generation. |
| `releaseId` | Stable, unique release identifier using lowercase letters, digits and hyphens. It is also the immutable GitHub release tag. |
| `amendment` | Display string from the reviewed manifest, such as `01-2026`. |
| `effectiveDate` | Effective calendar date from the reviewed manifest in ISO `YYYY-MM-DD` form. |
| `manifestSchemaVersion` | Positive integer. The first release uses chart manifest version `2`. |
| `manifestUrl` | Immutable direct HTTPS URL in this repository, under `/releases/download/<releaseId>/manifest.json`. |
| `manifestSha256` | SHA-256 of the exact manifest bytes, as 64 lowercase hexadecimal characters. |
| `totalPdfBytes` | Positive safe integer. Exact sum of the chart artifact lengths in bytes. |

Reject duplicate release IDs or sequences, unexpected dataset or malformed fields, invalid signatures, lower generations, and conflicting payload bytes for the same previously accepted generation. An exact repeat of an accepted generation can be reused. Keep the most recently authenticated index when a refresh fails. After successful refresh select the highest supported manifest-schema sequence above the local effective sequence. Omission from a newer valid generation retracts an offer. It does not request deletion of an installed snapshot. The publisher promotes releases on or after the recorded effective date.

## Download and verification

Fetch the exact manifest bytes and compare their SHA-256 digest before parsing the manifest. Confirm that its release ID, schema version, amendment, effective date, and total PDF bytes agree with the signed offer. Each manifest PDF artifact is available as `artifact-<id>` at the same immutable release path. Require a safe ID, exact byte count and matching SHA-256 for each asset. Validate the full inventory before switching to the new edition.

The initial request must use the exact configured HTTPS URL for this repository. GitHub release assets may redirect over HTTPS to GitHub-owned asset hosts such as `release-assets.githubusercontent.com`, `objects.githubusercontent.com` or `github-releases.githubusercontent.com`. Bound network requests and reject other redirect hosts. Recommended v1 limits are 64 KiB for the envelope, 1 MiB for the manifest, 256 PDFs, 128 MiB per PDF and 512 MiB for the snapshot, in addition to the tighter signed byte counts.

The bundled seed uses a commit-pinned release URL and manifest hash. The signed index serves runtime discovery independently of GitHub's repository-wide latest release.
