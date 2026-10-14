# adm.cloud.aws

An Amazon Web Services client in ADM over `std.net.http`: one module per service, and a root
module that holds credentials, Signature Version 4, regions, endpoints, retries and errors.
There is no SDK to install.

```bash
adm get admlang:adm.cloud.aws
```

```adm
use (
	adm.cloud.aws
	adm.cloud.aws.sqs
	adm.cloud.aws.secretsmanager
)

let account = try aws.connect("eu-central-1")        // the credential chain
let queue = new sqs.Client(account).queue("https://sqs.eu-central-1.amazonaws.com/123456789012/jobs")
try queue.send("resize 7.png", delay = 10s)
for let m in try queue.receive(10, wait = 20s) {
	print(m.body)
	try m.delete()
}

let secrets = new secretsmanager.Client(account)
let dbPassword = try secrets.get("prod/db/password")  // Secret<string>
```

## Credentials and endpoints

`aws.connect(region)` looks where the AWS tools look, in order: the environment
(`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`), the shared files
(`~/.aws/credentials`, `~/.aws/config`, `AWS_PROFILE`), a web identity token file (IRSA, EKS
Pod Identity), the container credentials endpoint (ECS, Lambda) and the instance metadata
service (IMDSv2). Temporary credentials renew before they expire. `aws.connect(region,
accessKey, secretKey)` takes them directly, and `assumeRole` makes an account for another
role.

Endpoints come from the region and the service; `endpoint` on the account overrides them for
local stacks (LocalStack, MinIO, DynamoDB Local) and for FIPS or dual-stack endpoints.

Every request is signed with Signature Version 4 (header or query form, the latter for presigned
URLs), retried on throttling and 5xx answers with backoff and a request limit, and fails with
`aws.AwsError`: `kind`, the service's error code, message and request id.

## Service modules

| Module | Covers |
|---|---|
| `s3` | buckets and objects, multipart uploads, presigned URLs, listing, versions, tags, lifecycle; MinIO, Ceph, Garage, R2 and B2 through `endpoint` |
| `sqs` | queues, send and receive with long polling, batches, visibility, dead-letter queues, FIFO groups and deduplication |
| `sns` | topics and subscriptions, publish (also batches, SMS, mobile push), verification of the signed messages SNS posts to HTTP subscribers |
| `dynamodb` | items as typed structs, get, put, update expressions, query and scan with pagination, batch and transactional writes, streams, tables and indexes |
| `secretsmanager` | get, put, list and rotate secrets, as `Secret<T>` |
| `ssm` | Parameter Store: get, put and list parameters by path, decrypted `SecureString` as `Secret<string>` |
| `sts` | assume role (also with web identity and SAML), session tokens, caller identity |
| `ses` | SES v2: send `std.formats.mail` messages and templates, suppression list, configuration sets, event notifications |
| `lambda` | invoke (synchronous, asynchronous, streamed responses), function management, and the runtime API so a function can be written in ADM |
| `cloudwatch` | metrics (put, get, alarms) and logs (groups, streams, put events, Logs Insights queries); an exporter for the Telemetry service |
| `kms` | encrypt, decrypt, sign, verify, generate data keys, key metadata |
| `eventbridge` | put events, rules and targets, schedules |
| `kinesis` | put records, shard iterators, enhanced fan-out consumers |
| `bedrock` | the runtime: converse and invoke with streaming, embeddings; the same client shape as the `adm.ai` libraries |

## Tests

`adm test cloud/aws` runs against a loopback server. `ADM_TEST_AWS=1` adds calls to the real
services with the usual credential chain and `AWS_REGION`, in resources named `adm-test-*` that
the suite creates and deletes.
