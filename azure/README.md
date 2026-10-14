# adm.cloud.azure

A Microsoft Azure client in ADM over `std.net.http`: one module per service, and a root module
that holds credentials, shared keys and SAS, clouds and endpoints, retries and errors. There is
no SDK to install.

**Not built yet.** The library is declared so that its name, modules and scope are fixed; this
README is the design the code will follow.

```bash
adm get admlang:adm.cloud.azure
```

```adm
use (
	adm.cloud.azure
	adm.cloud.azure.blob
	adm.cloud.azure.keyvault
)

let account = try azure.connect()                    // the credential chain
let photos = blob.Client(account, "mystorage").container("photos")
try photos.upload("2026/cat.jpg", pixels, "image/jpeg")
let link = try photos.sasURL("2026/cat.jpg", 1h)

let vault = keyvault.Client(account, "https://myvault.vault.azure.net")
let dbPassword = try vault.secret("db-password")     // Secret<string>
```

## Credentials and endpoints

`azure.connect()` looks where the Azure tools look: the environment (`AZURE_TENANT_ID`,
`AZURE_CLIENT_ID` with a secret or a certificate), workload identity (`AZURE_FEDERATED_TOKEN_FILE`),
managed identity (the IMDS endpoint, App Service, Container Apps) and the Azure CLI's token.
Tokens come from Entra ID (the v2 endpoints) per resource scope and renew before they expire.
Storage services also take a shared key or a SAS. `cloud` selects the public cloud, Azure
Government or Azure China endpoints.

Every request is retried on throttling and 5xx answers with backoff, and fails with
`azure.AzureError`: `kind`, the service's error code and message, and the request id.

## Service modules

| Module | Covers |
|---|---|
| `blob` | containers and blobs, block and append uploads, ranged downloads, SAS links, leases, conditions, tags, listing, soft delete |
| `queue` | enqueue, dequeue with visibility, peek, update, delete, queue metadata |
| `tables` | Table Storage and Cosmos DB tables: entities as typed structs, OData queries, entity batches |
| `servicebus` | queues, topics and subscriptions over AMQP 1.0 (spoken by this module; `adm.network.amqp` is AMQP 0-9-1): send, receive and peek-lock, sessions, dead-lettering, scheduled and deferred messages |
| `keyvault` | secrets, keys (encrypt, decrypt, wrap, sign, verify) and certificates |
| `cosmos` | Cosmos DB for NoSQL: items as typed structs, SQL queries with partition keys, transactional batches, the change feed |
| `monitor` | custom metrics and the Logs Ingestion API; an exporter for the Telemetry service |
| `openai` | Azure OpenAI: the `adm.ai.openai` client shape with Azure endpoints, deployments and Entra ID or key credentials |

## Builds on

`std.net.http`, `std.proto.auth.oauth2` and `std.proto.auth.jose` (Entra ID tokens, certificate
credentials), `std.crypto` (shared key signing), `std.data.encoding` (JSON, XML for the storage
services), `std.net.tcp` and `std.net.tls` for AMQP 1.0, `std.services.telemetry` for the
monitor exporter.

## Tests

`adm test cloud/azure` runs against a loopback server. `ADM_TEST_AZURE=1` adds calls to the real
services with the usual credential chain and `AZURE_RESOURCE_GROUP`, in resources named
`admtest*` that the suite creates and deletes.
