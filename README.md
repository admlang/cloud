# ADM cloud libraries

Official [ADM](https://github.com/admlang/adm) libraries that connect an ADM program to a cloud
provider. Each library is one cloud, written in ADM over `std.net.http`, published as
`adm.cloud.<cloud>` and signed by the `admlang` publisher. They live outside the standard
library because the clouds add and change operations every week.

| Library | Cloud | Service modules |
|---|---|---|
| [`adm.cloud.aws`](aws/) | Amazon Web Services | `s3`, `sqs`, `sns`, `dynamodb`, `secretsmanager`, `ssm`, `sts`, `ses`, `lambda`, `cloudwatch`, `kms`, `eventbridge`, `kinesis`, `bedrock` |
| [`adm.cloud.gcp`](gcp/) | Google Cloud | `storage`, `pubsub`, `firestore`, `secretmanager`, `bigquery`, `tasks`, `logging`, `kms`, `vertex` |
| [`adm.cloud.azure`](azure/) | Microsoft Azure | `blob`, `queue`, `tables`, `servicebus`, `keyvault`, `cosmos`, `monitor`, `openai` |
| [`adm.cloud.cloudflare`](cloudflare/) | Cloudflare | `dns`, `kv`, `d1`, `queues`, `workers`, `images`, `turnstile`, `ai` |

## One library per cloud, one module per service

A program imports the services it uses, and nothing else is compiled into it:

```adm
use (
	adm.cloud.aws
	adm.cloud.aws.sqs
	adm.cloud.aws.dynamodb
)
```

The root module of each library holds what every service shares: credentials and how they are
found, request signing, regions and endpoints, retries with backoff, pagination, and the one
error type (`aws.AwsError`, `gcp.GcpError`, `azure.AzureError`,
`cloudflare.CloudflareError`) with the service's code, message, request id and a `kind`
(`Denied`, `NotFound`, `Conflict`, `Throttled`, `Invalid`, `Transport`, `Server`).

A service module is a client over that root (`sqs.Client`, `storage.Client`), with methods
named after the service's operations, typed requests and results, `all()` on every paged
listing, and structs mapped onto the service's records where it keeps them (DynamoDB items,
Firestore documents, Cosmos items, Table entities).

Module names are the contract. Should a library grow too large to compile as one, it splits
into libraries of the same module names, and a program changes only its dependency list.

## Status

Nothing is built yet except S3, which exists today as `adm.network.s3` and moves here as
`adm.cloud.aws.s3`. Each library is declared so that its name, modules and scope are fixed, and
its README is the design the code will follow.

## Tests

Each suite runs against a loopback server that answers as the cloud does, so `adm test` needs
no account. With credentials the suite also calls the real services, in resources it creates
and deletes under a name prefix of its own:

```bash
ADM_TEST_AWS=1 adm test cloud/aws            # the usual AWS credential chain, AWS_REGION
ADM_TEST_GCP=1 adm test cloud/gcp            # application default credentials, a project
ADM_TEST_AZURE=1 adm test cloud/azure        # the usual Azure credential chain, a resource group
ADM_TEST_CLOUDFLARE_TOKEN=... adm test cloud/cloudflare
```

Each library's README says what the account must hold.
