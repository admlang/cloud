# adm.cloud.cloudflare

A Cloudflare API client in ADM over `std.net.http`: one module per product, and a root module
that holds API tokens, accounts and zones, the v4 API envelope, pagination, retries and errors.

**Not built yet.** The library is declared so that its name, modules and scope are fixed; this
README is the design the code will follow.

```bash
adm get admlang:adm.cloud.cloudflare
```

```adm
use (
	adm.cloud.cloudflare
	adm.cloud.cloudflare.kv
	adm.cloud.cloudflare.dns
)

let cf = try cloudflare.connect(token)               // CLOUDFLARE_API_TOKEN when left out
let settings = kv.Client(cf.account("0123abcd")).namespace("settings")
try settings.put("theme", "dark", expire = 7d)
let theme = try settings.get("theme")                // ?string

let zone = try cf.zone("example.com")
try dns.Client(zone).create("A", "www", "203.0.113.7", proxied = true)
```

## Tokens and scoping

`cloudflare.connect(token)` takes an API token (a `Secret<string>`, or `CLOUDFLARE_API_TOKEN`);
`connect(email, key)` takes the global key for the few endpoints that still need it. An
`Account` and a `Zone` scope the product clients. Every response is unwrapped from the v4
envelope, its `errors` become `cloudflare.CloudflareError` (`kind`, code, message, the ray id),
listings page through `all()`, and requests are retried on `429` and 5xx answers with backoff.

R2 is used through `adm.cloud.aws.s3` with the account's R2 endpoint and an R2 token; this
library adds nothing for it.

## Product modules

| Module | Covers |
|---|---|
| `dns` | records: list, create, update, delete, batch; zone import and export; DNSSEC settings |
| `kv` | namespaces, get and put with metadata and expiry, bulk writes and deletes, list keys with prefixes |
| `d1` | databases: query with bound parameters, batches, raw results, exports; a `std.data.db` driver for the `d1:` scheme |
| `queues` | send and batch send, pull consumers, acknowledge and retry |
| `workers` | scripts and versions: upload with bindings, deploy and rollback, routes and custom domains, secrets, tail logs over WebSocket |
| `images` | upload by file or URL, variants, signed delivery URLs, direct creator upload |
| `turnstile` | server-side verification of a widget response (siteverify), with idempotency keys |
| `ai` | Workers AI: run text, embedding, image and speech models by name, with streaming; the same client shape as the `adm.ai` libraries |

## Builds on

`std.net.http` (client, WebSocket for Worker tails), `std.data.encoding` (JSON), `std.data.db`
for the D1 driver, `std.ai.llm` message types for Workers AI.

## Tests

`adm test cloud/cloudflare` runs against a loopback server. `ADM_TEST_CLOUDFLARE_TOKEN` and
`ADM_TEST_CLOUDFLARE_ZONE` add calls to the real API, in records, namespaces and databases
named `adm-test-*` that the suite creates and deletes.
