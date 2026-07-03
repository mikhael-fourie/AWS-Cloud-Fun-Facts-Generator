# AI-Enhanced Serverless Web App ☁️

A small serverless web app that serves cloud-computing trivia, rewritten on the fly to be witty by a generative AI model. Built to get hands-on with a multi-service AWS architecture — API Gateway, Lambda, DynamoDB, and Bedrock — behind a lightweight static frontend.

**Status:** Feature-complete. The live AWS infrastructure has since been decommissioned to avoid ongoing costs; the code here reflects the working project as it was deployed.

## How it works

1. The frontend (a single static HTML/CSS/JS page) sends a `GET` request to an API Gateway endpoint when the user clicks "Generate Fun Fact."
2. API Gateway triggers an AWS Lambda function (Python).
3. The Lambda function scans a DynamoDB table (`CloudFacts`) and picks a random stored fact.
4. That fact is sent to Amazon Bedrock, which invokes Claude 3.5 Sonnet with a prompt asking it to rewrite the fact as a short, witty 1–2 sentence version.
5. The rewritten fact is returned as JSON; if Bedrock fails or the output looks too long, the Lambda falls back to the original, unmodified fact instead of erroring out.
6. The frontend displays the result, with loading and error states handled in the UI.

## Architecture

```
Browser (static HTML/CSS/JS)
      │  GET request
      ▼
Amazon API Gateway (REST endpoint)
      │  invokes
      ▼
AWS Lambda (Python)
      │                     │
      ▼                     ▼
Amazon DynamoDB        Amazon Bedrock
(stores raw facts)     (Claude 3.5 Sonnet —
                         rewrites the fact)
```

Frontend was hosted as a static site via AWS Amplify. Lambda's execution role was scoped to DynamoDB read access (`Scan`) and Bedrock model invocation (`InvokeModel`) — no broader permissions than the function actually uses.

## Tech stack

|Layer|Service|Purpose|
|---|---|---|
|Frontend|HTML / CSS / vanilla JavaScript|UI, fetch call to the API|
|Hosting|AWS Amplify|Static site hosting|
|API|Amazon API Gateway|REST endpoint, CORS-enabled|
|Compute|AWS Lambda (Python, boto3)|Business logic|
|Database|Amazon DynamoDB|Stores the raw fact pool|
|AI|Amazon Bedrock (Claude 3.5 Sonnet)|Rewrites facts to be witty|
|Access control|AWS IAM|Least-privilege role for the Lambda function|

## Project structure

```
├── index.html          # Frontend: UI, fetch logic, loading/error states
└── lambda_function.py  # Backend: DynamoDB scan, Bedrock invoke, response handling
```

## Notable implementation details

- **Graceful degradation:** if Bedrock is unavailable or returns something malformed, the Lambda silently falls back to the original fact rather than returning an error to the user.
- **CORS handled explicitly** in the Lambda response headers, since API Gateway and Amplify are on different origins.
- **No secrets committed:** the frontend's `API_URL` is left as a placeholder (`YOUR-API-ID`) — replace it with your own API Gateway invoke URL if redeploying.

## Redeploying this yourself

This was originally set up manually through the AWS Console rather than with an IaC tool, so there's no `template.yaml` or Terraform config here. To recreate it:

1. Create a DynamoDB table named `CloudFacts` with a `FactText` attribute on each item, and seed it with some facts.
2. Create the Lambda function from `lambda_function.py`, with an execution role granting `dynamodb:Scan` on the table and `bedrock:InvokeModel` for the Claude 3.5 Sonnet model.
3. Request Bedrock model access for Claude 3.5 Sonnet in your AWS account (this requires explicit enablement).
4. Create a REST API in API Gateway with a `GET` method integrated to the Lambda function, and enable CORS.
5. Update `API_URL` in `index.html` with your API Gateway invoke URL.
6. Deploy `index.html` via AWS Amplify (or any static host).
