---
layout: single
title: "Triggering Pipelines with Webhooks"
sidebar:
  nav: guides
redirect_from: /guides/user/triggers/webhooks/
---

{% include toc %}

## Overview

Spinnaker allows you to programmatically trigger pipelines using webhooks. By sending a `POST` request to a predefined Spinnaker endpoint, you can start a pipeline when:

- A CI/CD job completes
- A third-party system triggers an event
- A manual request is made via the command line

The webhook payload—whether custom-defined or provided by an external service—will be accessible within the pipeline execution.

> **Note:** You can configure multiple pipelines to trigger from a single webhook.

If you're using a *GitHub* webhook, follow the [GitHub trigger setup guide](/setup/triggers/github/).

If your Spinnaker instance has authentication enabled, refer to the [Automated Pipeline Triggers setup](/setup/security/authorization/#automated-pipeline-triggers).

## Adding a Webhook Trigger to a Pipeline

To add a webhook trigger to a pipeline:

1. Open your pipeline in Spinnaker.
2. Navigate to **Configuration**.
3. Click **Add Trigger**.
4. Set the **Type** to **Webhook**.
5. Enter a unique value in the **Source** field.

Spinnaker will generate an endpoint for the webhook, which you can use to trigger the pipeline.

{%
  include
  figure
  image_path="./basic-webhook.png"
%}

In the example above, the pipeline can be triggered by sending a `POST` request to:

```
http://localhost:8084/webhooks/webhook/demo
```

If your Spinnaker instance is running on a different domain (e.g., `https://api.spinnaker-prod.net`), the correct URL will be displayed.

### Triggering the Pipeline via API

Save the pipeline configuration and trigger it using:

```bash
curl $ENDPOINT -X POST -H "Content-Type: application/json" -d "{}"
```

Replace `$ENDPOINT` with the actual webhook URL generated for your pipeline.

## Payload Constraints

You can restrict a webhook trigger to specific payload conditions using **Payload Constraints**. These constraints define key-value pairs where:

- The key must be present in the incoming payload.
- The value must match a regex pattern.

For example, with the following constraints:

{%
  include
  figure
  image_path="./constraints-webhook.png"
  caption="Payload constraints: `foo = bar` and `bing = b.*p`."
%}

This payload **will trigger** the pipeline:

```json
{
  "foo": "bar",
  "bing": "boooop",
  "x": ["1", "2", "3"]
}
```

This payload **will not trigger** the pipeline:

```json
{
  "foo": "bar",
  "x": ["1", "2", "3"]
}
```

## Passing Parameters

If your pipeline accepts parameters (e.g., selecting a deployment stack), define them under **Pipeline Parameters** in the webhook trigger configuration:

{%
  include
  figure
  image_path="./parameters.png"
  caption="Refer to the [Pipeline Expressions Guide](/guides/user/pipeline-expressions) for more details."
%}

When executing the pipeline manually, Spinnaker will prompt for parameter values:

{%
  include
  figure
  image_path="./manual-execution.png"
%}

### Supplying Parameters via Webhook

When triggering a pipeline through a webhook, include parameters inside a `parameters` key in the payload:

```json
{
  "parameters": {
    "stack": "prod"
  }
}
```

> **Note:** If a parameter is marked **Required** without a default value, the pipeline will fail if the parameter is missing.

## Passing Artifacts

If your pipeline requires artifacts (e.g., Kubernetes manifests in GCS), define them under **Expected Artifacts** in the webhook configuration:

{%
  include
  figure
  image_path="./artifacts.png"
%}

To pass an artifact in a webhook payload, use the `artifacts` list:

```json
{
  "artifacts": [
    {
      "type": "gcs/object",
      "name": "manifest.yml",
      "reference": "gs://lw-artifacts/manifest.yml"
    }
  ]
}
```

## Testing Webhooks

Before integrating webhooks into your workflow, test them using API request tools like:

- **[Beeceptor](https://beeceptor.com/)** - Simulate and inspect HTTP requests.
- **[Webhook.site](https://webhook.site/)** - Capture and debug webhook payloads.

These tools help verify your webhook payloads and confirm pipeline triggers work as expected.
