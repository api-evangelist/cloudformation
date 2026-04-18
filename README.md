# AWS CloudFormation (cloudformation)

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/cloudformation/refs/heads/main/apis.yml)

A collection of APIs provided by AWS for infrastructure as code provisioning and management of AWS and third-party resources using CloudFormation templates and the Cloud Control API.

## Timestamps

- **Created:** 2024
- **Modified:** 2026-04-18

## APIs

### AWS CloudFormation API
AWS CloudFormation gives you an easy way to model a collection of related AWS and third-party resources, provision them quickly and consistently, and manage them throughout their lifecycles. It uses templates to define stacks of resources and provides API operations for creating, updating, and deleting stacks.

**Human URL:** [https://aws.amazon.com/cloudformation/](https://aws.amazon.com/cloudformation/)

#### Tags:

 - Infrastructure, Resources, Stacks, Templates

#### Properties

- [Documentation](https://docs.aws.amazon.com/cloudformation/)
- [APIReference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/cloudformation-api.yml)
- [Pricing](https://aws.amazon.com/cloudformation/pricing/)
- [GettingStarted](https://aws.amazon.com/cloudformation/getting-started/)
- [ChangeLog](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/document-history.html)
- [SDK - Python (Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudformation.html)

### AWS Cloud Control API
AWS Cloud Control API provides a uniform CRUDL (create, read, update, delete, list) interface for managing AWS and third-party resources. It offers a standardized way to access and provision resource types available in the CloudFormation Registry without needing to learn each individual service API.

**Human URL:** [https://aws.amazon.com/cloudcontrolapi/](https://aws.amazon.com/cloudcontrolapi/)

#### Tags:

 - Cloud Control, CRUDL, Provisioning, Resources

#### Properties

- [Documentation](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/what-is-cloudcontrolapi.html)
- [OpenAPI](openapi/cloud-control-api.yml)
- [APIReference](https://docs.aws.amazon.com/cloudcontrolapi/index.html)

## Common Properties

- [Portal](https://aws.amazon.com/cloudformation/resources/)
- [GettingStarted](https://aws.amazon.com/cloudformation/getting-started/)
- [Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
- [Pricing](https://aws.amazon.com/cloudformation/pricing/)
- [TermsOfService](https://aws.amazon.com/terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Blog](https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/)
- [StatusPage](https://status.aws.amazon.com/)
- [Console](https://console.aws.amazon.com/cloudformation/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [GitHubOrganization](https://github.com/aws-cloudformation)
- [GitHubRepository](https://github.com/aws-cloudformation/aws-cloudformation-templates)
- [StackOverflow](https://stackoverflow.com/questions/tagged/aws-cloudformation)
- [SDK - Python (Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudformation.html)
- [SDK - JavaScript v3](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/cloudformation-examples.html)
- [SDK - Java](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/examples-cloudformation.html)
- [SDK - Go](https://docs.aws.amazon.com/sdk-for-go/api/service/cloudformation/)
- [CLI - AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/)
- [CLI - Rain](https://github.com/aws-cloudformation/rain)

## OpenAPI

- [CloudFormation API OpenAPI](openapi/cloudformation-api.yml)
- [Cloud Control API OpenAPI](openapi/cloud-control-api.yml)

## JSON Schema

| Schema | File |
|---|---|
| Stack Schema | [json-schema/stack.json](json-schema/stack.json) |
| Template Schema | [json-schema/template.json](json-schema/template.json) |
| Resource Schema | [json-schema/resource.json](json-schema/resource.json) |
| Change Set Schema | [json-schema/change-set.json](json-schema/change-set.json) |
| Stack Event Schema | [json-schema/cloudformation-stack-event-schema.json](json-schema/cloudformation-stack-event-schema.json) |
| Stack Summary Schema | [json-schema/cloudformation-stack-summary-schema.json](json-schema/cloudformation-stack-summary-schema.json) |
| Cloud Control Progress Event | [json-schema/cloud-control-progress-event-schema.json](json-schema/cloud-control-progress-event-schema.json) |
| Cloud Control Resource Description | [json-schema/cloud-control-resource-description-schema.json](json-schema/cloud-control-resource-description-schema.json) |

## JSON-LD

- [JSON-LD Context](json-ld/context.jsonld)
- [CloudFormation Context](json-ld/cloudformation-context.jsonld)
- [Cloud Control Context](json-ld/cloud-control-context.jsonld)

## Vocabulary

- [CloudFormation Vocabulary](vocabulary/cloudformation-vocabulary.yaml)

## Rules

- [Spectral Rules](rules/cloudformation-spectral-rules.yml)

## Capabilities

| Capability | Type | File |
|---|---|---|
| Infrastructure Provisioning | Workflow | [capabilities/infrastructure-provisioning.yaml](capabilities/infrastructure-provisioning.yaml) |
| CloudFormation | Shared | [capabilities/shared/cloudformation.yaml](capabilities/shared/cloudformation.yaml) |
| Cloud Control | Shared | [capabilities/shared/cloud-control.yaml](capabilities/shared/cloud-control.yaml) |

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
