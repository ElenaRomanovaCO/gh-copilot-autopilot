# Calling Claude on Bedrock from outside AWS, with Strands

The problem this solves: you have a Strands agent running somewhere that is not AWS — your laptop, a VPS, Vercel, a container on someone else's cloud — and you want it to talk to Claude on Amazon Bedrock without shipping AWS credentials or configuring an IAM credential chain on that host.

The answer is an **Amazon Bedrock API key**: a bearer token you put in one environment variable. Bedrock's endpoints are on the public internet, so nothing else is required. No VPC, no `aws configure`, no credentials file.

Everything below was verified against a live account, not assumed.

---

## The short version

```python
import os
os.environ["AWS_BEARER_TOKEN_BEDROCK"] = "<your key>"   # BEFORE the model is constructed

from strands import Agent
from strands.models import BedrockModel

model = BedrockModel(
    model_id="us.anthropic.claude-sonnet-4-6",
    region_name="us-east-1",
    max_tokens=4096,
    temperature=0.3,
    streaming=True,
)

agent = Agent(model=model)
print(agent("Hello"))
```

That is the whole integration. The rest of this file is where the key comes from and what goes wrong.

---

## Why this works, given that Strands never mentions it

Strands' documentation does not mention `AWS_BEARER_TOKEN_BEDROCK` anywhere. It still works, and the reason matters for debugging: **the support lives in botocore, not in Strands.**

Botocore resolves an `AWS_BEARER_TOKEN_<SIGNING_NAME>` variable generically — see `get_token_from_environment` in `botocore/utils.py`. Bedrock's signing name is `bedrock`, so the variable is `AWS_BEARER_TOKEN_BEDROCK`. Any library that builds a boto3 `bedrock-runtime` client inherits the behaviour for free, and `BedrockModel` builds exactly such a client.

Consequence: if a future Strands release changes how it constructs its client, this keeps working as long as it still goes through boto3. And when it breaks, botocore is where to look, not Strands.

Requires a botocore recent enough to have bearer-token support (1.39 or later; anything installed alongside a current `strands-agents` is well past that).

### Proof

Minting a real short-term token, then deleting **every** normal AWS credential from the environment — access key, secret, session token, profile, shared credentials file, config file, IMDS — and leaving only the bearer token:

```
cleared AWS credentials; only AWS_BEARER_TOKEN_BEDROCK remains
boto3   -> BOTO3-OK
strands -> STRANDS-OK
```

Both paths answered on a host with no AWS identity of any kind.

---

## Getting a key

Two kinds, and the difference is not cosmetic.

| | Short-term | Long-term |
|---|---|---|
| Lifetime | 12 hours maximum, hard cap | Your choice, including never |
| Backed by | The IAM principal that generated it | A dedicated IAM user |
| AWS's position | Recommended for production | "Exploration only" |

### Short-term

Console → Amazon Bedrock → **API keys** → *Short-term API keys* → Generate. Expires with your console session, 12 hours at the outside.

Or generate one locally from ambient IAM credentials:

```bash
pip install aws-bedrock-token-generator
```

```python
from aws_bedrock_token_generator import provide_token
token = provide_token(region="us-east-1")
```

This is the honest production answer *if the host has AWS credentials to mint from*. For a genuinely external host that has none, it solves nothing — which is why the long-term key exists.

### Long-term, never expiring

The console offers a dropdown of preset expirations. The IAM API does not have that limit. From the `CreateServiceSpecificCredential` reference, `CredentialAgeDays` is optional, valid range 1–36,600 days, and:

> When not specified, the credential will not expire.

So omit the flag entirely:

```bash
# 1. A dedicated user that exists only to hold this key
aws iam create-user --user-name strands-bedrock-external

# 2. Least privilege — see the policy below, NOT AmazonBedrockLimitedAccess
aws iam put-user-policy \
  --user-name strands-bedrock-external \
  --policy-name BedrockInvokeClaude \
  --policy-document file://bedrock-invoke-policy.json

# 3. The key. No --credential-age-days means no expiry.
aws iam create-service-specific-credential \
  --user-name strands-bedrock-external \
  --service-name bedrock.amazonaws.com
```

`ServiceApiKeyValue` in the response is the key. **It is shown exactly once** and cannot be recovered — only reset. Keep `ServiceSpecificCredentialId` too; that is the handle for disabling or deleting it later.

---

## The IAM policy

Do not attach `AmazonBedrockLimitedAccess`. It is much broader than "call one model", and a never-expiring bearer token deserves a narrow blast radius.

Calling a model through a cross-region inference profile needs **two** kinds of permission, which is the single most common reason a correct-looking policy returns `AccessDenied`: the profile ARN *and* the foundation-model ARN in **every region the profile routes to**. Check the fan-out before writing the policy:

```bash
aws bedrock get-inference-profile \
  --region us-east-1 \
  --inference-profile-identifier us.anthropic.claude-sonnet-4-6
```

`bedrock-invoke-policy.json`, for Sonnet 4.6 and Opus 4.5:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "InvokeClaudeSonnet46AndOpus45",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:us-east-1:<ACCOUNT_ID>:inference-profile/us.anthropic.claude-sonnet-4-6",
        "arn:aws:bedrock:us-east-1:<ACCOUNT_ID>:inference-profile/us.anthropic.claude-opus-4-5-20251101-v1:0",
        "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-sonnet-4-6",
        "arn:aws:bedrock:us-east-2::foundation-model/anthropic.claude-sonnet-4-6",
        "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-sonnet-4-6",
        "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-opus-4-5-20251101-v1:0",
        "arn:aws:bedrock:us-east-2::foundation-model/anthropic.claude-opus-4-5-20251101-v1:0",
        "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-opus-4-5-20251101-v1:0"
      ]
    }
  ]
}
```

Note the account ID appears on inference-profile ARNs and is empty on foundation-model ARNs. That is correct, not a typo.

`Converse` and `ConverseStream` are authorised by `InvokeModel` and `InvokeModelWithResponseStream` — there is no separate `bedrock:Converse` action to add.

Worth adding if it fits your deployment:

- **`aws:SourceIp`** condition, if the external host has a stable egress IP. This is the single highest-value restriction on a non-expiring key.
- **A budget alarm** on the account, so a leaked key shows up as an alert rather than as an invoice.

---

## Which models you can actually reach

Listed is not the same as usable, and the failure modes are genuinely different. Probe them — a five-token `converse` call per model costs approximately nothing and answers the question definitively:

```bash
aws bedrock-runtime converse --region us-east-1 \
  --model-id us.anthropic.claude-sonnet-4-6 \
  --messages '[{"role":"user","content":[{"text":"hi"}]}]' \
  --inference-config '{"maxTokens":5}'
```

The three denials you will meet, and what each actually means:

| Error text | Meaning | Fixable by you? |
|---|---|---|
| `not authorized to perform the required AWS Marketplace actions` | Your principal lacks `aws-marketplace:ViewSubscriptions` / `Subscribe` | Yes — add the permissions, subscribe, retry after ~2 min |
| `X is not available for this account` | Account-level entitlement missing | No — AWS Sales |
| `marked by provider as Legacy and you have not been actively using the model` | Auto-disabled after 30 days idle | Use a current model |

Only the first is a permissions problem. The second looks identical from the client and is not.

---

## Things that will cost you an hour

- **Set the environment variable before constructing `BedrockModel`.** Botocore reads it at client-creation time. Setting it afterwards produces `NoCredentialsError`, which reads like a completely unrelated failure. In production set it in the real environment — systemd unit, Docker `-e`, your platform's secret store — not in Python.
- **Strands defaults to `us-west-2`, not `us-east-1`.** Your key is scoped to the region it was generated in. Pass `region_name` explicitly every time, or use a `global.` profile ID (`global.anthropic.claude-sonnet-4-6`) that routes anywhere.
- **Inference-profile IDs only.** Current Claude models on Bedrock are `INFERENCE_PROFILE`-only — there is no bare `modelId` invoke path. Use the `us.` or `global.` prefixed ID exactly as `list-inference-profiles` reports it. Some carry a `-v1` or dated suffix and some do not; do not pattern-match, copy.
- **Bedrock actions only.** The key does not work with Bedrock Agents, Data Automation, or `InvokeModelWithBidirectionalStream`.
- **Two keys per user per service, maximum.** Need more, or want per-project isolation? More users.
- **An org SCP may block a never-expiring key.** AWS publishes a policy example built on the `iam:ServiceSpecificCredentialAgeDays` condition key that caps credential age. If your org uses one, the no-expiry call is denied outright — you find out immediately.

---

## Rotation and revocation

A never-expiring key has no lifecycle of its own, so give it one deliberately.

```bash
# What exists
aws iam list-service-specific-credentials \
  --user-name strands-bedrock-external \
  --service-name bedrock.amazonaws.com

# Turn it off without destroying it — reversible, good first move on suspicion
aws iam update-service-specific-credential \
  --user-name strands-bedrock-external \
  --service-specific-credential-id <ID> \
  --status Inactive

# Rotate: new secret, same credential ID
aws iam reset-service-specific-credential \
  --user-name strands-bedrock-external \
  --service-specific-credential-id <ID>

# Permanent
aws iam delete-service-specific-credential \
  --user-name strands-bedrock-external \
  --service-specific-credential-id <ID>
```

Every call is logged in CloudTrail. The key travels in the `Authorization` header and is **not** logged.

---

## When to use something else

This document is about Bedrock specifically. Two neighbours are easy to confuse with it:

- **Claude Platform on AWS** — Anthropic-operated, still AWS IAM auth and Marketplace billing, but same-day model parity and bare model IDs (`claude-opus-5`). It does not go through Bedrock's per-model entitlement process, so it is the better route when the model you want is not available to your Bedrock account. Different client (`AnthropicAWS`), needs `AWS_REGION` and `ANTHROPIC_AWS_WORKSPACE_ID`.
- **The first-party Anthropic API** — if AWS is not actually a requirement, an `sk-ant-…` key is simpler than all of the above and every model is available on day one.
