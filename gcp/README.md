# adm.cloud.gcp

A Google Cloud client in ADM over `std.net.http`: one module per service, and a root module that
holds credentials, OAuth2 tokens, projects and locations, retries and errors. There is no SDK to
install.

**Not built yet.** The library is declared so that its name, modules and scope are fixed; this
README is the design the code will follow.

```bash
adm get admlang:adm.cloud.gcp
```

```adm
use (
	adm.cloud.gcp
	adm.cloud.gcp.storage
	adm.cloud.gcp.pubsub
)

let project = try gcp.connect("my-project")          // application default credentials
let photos = storage.Client(project).bucket("photos")
try photos.put("2026/cat.jpg", pixels, "image/jpeg")

let topic = pubsub.Client(project).topic("orders")
try topic.publish("order 7 paid", orderingKey = "7")
```

## Credentials and endpoints

`gcp.connect(project)` looks where the Google tools look: `GOOGLE_APPLICATION_CREDENTIALS`, the
gcloud application default credentials file, the metadata server (Compute Engine, GKE, Cloud
Run, Cloud Functions) and workload identity federation files. A service account key becomes a
signed JWT exchanged for an access token, or used as a self-signed token where the service
allows it; `connect(project, keyFile)` takes a key directly, and `impersonate` acts as another
service account. Tokens renew before they expire.

Every request is retried on `429`, `502`, `503` and `504` answers with backoff, and fails with
`gcp.GcpError`: `kind`, the service's status, message and reason, and the request id.

## Service modules

| Module | Covers |
|---|---|
| `storage` | buckets and objects, resumable uploads, ranged downloads, signed URLs (V4), preconditions, listing, lifecycle, notifications |
| `pubsub` | topics and subscriptions, publish with ordering keys and batching, pull and streaming pull, acknowledgement deadlines and extension, dead-letter topics, schemas |
| `firestore` | documents as typed structs, queries and aggregations, transactions, batched writes, listeners over the streaming endpoint |
| `secretmanager` | access, add, list and disable secret versions, as `Secret<T>` |
| `bigquery` | queries as jobs with typed rows and pagination, parameters, streaming inserts and the Storage Write API, datasets and tables |
| `tasks` | queues and HTTP tasks with schedule times and OIDC tokens |
| `logging` | write log entries and custom metrics to Cloud Logging and Monitoring; an exporter for the Telemetry service |
| `kms` | encrypt, decrypt, sign, verify, key rings and versions |
| `vertex` | Gemini and partner models with Google Cloud credentials: the same client shape as the `adm.ai` libraries |

## Builds on

`std.net.http`, `std.proto.auth.jose` (service account JWTs), `std.proto.auth.oauth2`,
`std.crypto`, `std.data.encoding` (JSON), `std.net.grpc` where a service is better served by
gRPC, `std.services.telemetry` for the logging exporter.

## Tests

`adm test cloud/gcp` runs against a loopback server. `ADM_TEST_GCP=1` adds calls to the real
services with application default credentials and `GOOGLE_CLOUD_PROJECT`, in resources named
`adm-test-*` that the suite creates and deletes.
