# AWS CloudFormation (cloudformation)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

A collection of APIs provided by AWS for infrastructure as code provisioning and management of AWS and third-party resources using CloudFormation templates and the Cloud Control API.

**APIs.json:** [https://docs.aws.amazon.com/cloudformation/](https://docs.aws.amazon.com/cloudformation/)

## Tags

- Automation
- AWS
- Cloud Resources
- IaC
- Infrastructure As Code
- Stack Management

## Timestamps

- **Created:** 2024
- **Modified:** 2026-05-19

## APIs

### AWS CloudFormation API

AWS CloudFormation gives you an easy way to model a collection of related AWS and third-party resources, provision them quickly and consistently, and manage them throughout their lifecycles. It uses templates to define stacks of resources and provides API operations for creating, updating, and deleting stacks.

- **Human URL:** [https://aws.amazon.com/cloudformation/](https://aws.amazon.com/cloudformation/)
- **Base URL:** `https://cloudformation.{region}.amazonaws.com`

#### Tags

- Infrastructure
- Resources
- Stacks
- Templates

#### Properties

- [Documentation](https://docs.aws.amazon.com/cloudformation/)
- [API Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/cloudformation-api.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/cloudformation-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cloudformation-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Pricing](https://aws.amazon.com/cloudformation/pricing/)
- [Getting Started](https://aws.amazon.com/cloudformation/getting-started/)
- [Changelog](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/document-history.html)
- [SDK](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudformation.html)

### AWS Cloud Control API

AWS Cloud Control API provides a uniform CRUDL (create, read, update, delete, list) interface for managing AWS and third-party resources. It offers a standardized way to access and provision resource types available in the CloudFormation Registry without needing to learn each individual service API.

- **Human URL:** [https://aws.amazon.com/cloudcontrolapi/](https://aws.amazon.com/cloudcontrolapi/)
- **Base URL:** `https://cloudcontrolapi.{region}.amazonaws.com`

#### Tags

- Cloud Control
- CRUDL
- Provisioning
- Resources

#### Properties

- [Documentation](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/what-is-cloudcontrolapi.html)
- [OpenAPI](openapi/cloud-control-api.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/cloud-control-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cloud-control-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [API Reference](https://docs.aws.amazon.com/cloudcontrolapi/index.html)

## Common Properties

- [Arazzo Workflows](arazzo/) — [Arazzo Specification](https://spec.openapis.org/arazzo/latest.html)
- [Portal](https://aws.amazon.com/cloudformation/resources/)
- [Getting Started](https://aws.amazon.com/cloudformation/getting-started/)
- [Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
- [Pricing](https://aws.amazon.com/cloudformation/pricing/)
- [Terms of Service](https://aws.amazon.com/terms/)
- [Privacy Policy](https://aws.amazon.com/privacy/)
- [Blog](https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/)
- [Status Page](https://status.aws.amazon.com/)
- [Console](https://console.aws.amazon.com/cloudformation/)
- [Sign Up](https://portal.aws.amazon.com/billing/signup)
- [GitHub Organization](https://github.com/aws-cloudformation)
- [GitHub Repository](https://github.com/aws-cloudformation/aws-cloudformation-templates)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/aws-cloudformation)
- [Features](undefined)
- [Use Cases](undefined)
- [Integrations](undefined)
- [SDK](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudformation.html)
- [SDK](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/cloudformation-examples.html)
- [SDK](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/examples-cloudformation.html)
- [SDK](https://docs.aws.amazon.com/sdk-for-go/api/service/cloudformation/)
- [C L I](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/)
- [C L I](https://github.com/aws-cloudformation/rain)
- [JSON Schema](json-schema/stack.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/template.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/resource.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/change-set.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-change-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-change-set-detail-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-change-set-summary-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-error-response-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-stack-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-stack-event-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-stack-resource-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-stack-summary-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloudformation-tag-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloud-control-progress-event-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloud-control-resource-description-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cloud-control-error-response-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON-LD](json-ld/context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [JSON-LD](json-ld/cloudformation-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [JSON-LD](json-ld/cloud-control-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [Vocabulary](vocabulary/cloudformation-vocabulary.yaml)
- [Rules](rules/cloudformation-spectral-rules.yml)
- [Capabilities](capabilities/infrastructure-provisioning.yaml)
- [Capabilities](capabilities/shared/cloudformation.yaml)
- [Capabilities](capabilities/shared/cloud-control.yaml)
- [M C P Server](https://aws.amazon.com/blogs/devops/introducing-the-aws-infrastructure-as-code-mcp-server-ai-powered-cdk-and-cloudformation-assistance/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
