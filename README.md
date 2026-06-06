# nodejs-cicd-app

CI/CD pipeline using GitHub Actions with OIDC to upload PDF books to an AWS S3 bucket.

## Books

- BTA-CICD-LECTURE-NOTES.pdf
- Linux-book.pdf
- Linux-Cheat-Sheet-Sponsored-By-Loggly.pdf

## Setup

### 1. AWS OIDC Identity Provider

In IAM > Identity providers, add:
- Provider URL: `https://token.actions.githubusercontent.com`
- Audience: `sts.amazonaws.com`

### 2. IAM Role

Create a role with the GitHub OIDC trust policy:

```json
{
  "Effect": "Allow",
  "Principal": { "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com" },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" },
    "StringLike": { "token.actions.githubusercontent.com:sub": "repo:<YOUR_ORG>/<YOUR_REPO>:*" }
  }
}
```

Attach a policy allowing `s3:PutObject` on your target bucket.

### 3. GitHub Secrets

Set the following repository secrets:

| Secret | Description |
|--------|-------------|
| `AWS_ROLE_ARN` | IAM role ARN |
| `AWS_REGION` | e.g. `us-east-1` |
| `S3_BUCKET_NAME` | Target S3 bucket name |

## Pipeline

On push to `main`, the workflow authenticates via OIDC and uploads all three PDFs to S3.
