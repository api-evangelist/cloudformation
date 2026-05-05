---
title: "How to Simplify Multi-Account Deployments Monitoring: Centralized Logs for AWS CloudFormation StackSets"
url: "https://aws.amazon.com/blogs/devops/how-to-simplify-multi-account-deployments-monitoring-centralized-logs-for-aws-cloudformation-stacksets/"
date: "Fri, 24 Oct 2025 15:35:20 +0000"
author: "Idriss Laouali Abdou"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<h1>Introduction</h1> 
<p>As organizations adopt multi-account strategies for&nbsp;improved security features and governance, <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html">AWS CloudFormation StackSets</a> enables organizations to deploy infrastructure across multiple accounts and regions. However, monitoring and tracking these distributed deployments across multiple accounts&nbsp;presents operational challenges. When a critical security baseline deployed across 50 accounts suddenly starts failing, teams face the daunting task of logging into each account individually to&nbsp;understand what went wrong and which accounts were affected.</p> 
<p>This operational overhead scales exponentially with organization growth,&nbsp;requiring platform teams to spend countless hours switching between accounts and manually correlating deployment events. The lack of centralized visibility slows incident response and makes it&nbsp;difficult to identify patterns or implement proactive monitoring.&nbsp;In this blog post, we’ll explore a solution that centralizes AWS CloudFormation logs from multiple accounts into a single management account, making it easier to monitor and troubleshoot StackSets deployments.</p> 
<h2>Solution Architecture</h2> 
<p>Our solution creates a centralized logging system that collects AWS CloudFormation events from all target accounts and forwards them to a central management account. This approach provides a single pane of glass for monitoring and troubleshooting AWS CloudFormation deployments across your entire organization.</p> 
<h2><img alt="Figure 1.&nbsp;Architecture diagram showing event flow from member accounts to management account through EventBridge and CloudWatch Logs" class="aligncenter wp-image-24102 size-large" height="625" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/21/StackSets-monitoring-architecture-diagram-1024x625.png" width="1024" /></h2> 
<p><strong>Figure 1. Architecture diagram showing event flow from member accounts to management account through EventBridge and CloudWatch Logs.</strong></p> 
<p>The architecture consists of four main components:</p> 
<ol> 
 <li><strong>Management Account Setup</strong>: Creates a central event bus, log group, and necessary permissions in the organization’s management account.</li> 
 <li><strong>Target Account Configuration</strong>: Deployed via StackSets to configure event rules that forward AWS CloudFormation events to the management account.</li> 
 <li><strong>Resource Deployment:</strong> Uses StackSets to deploy common resources across target accounts, generating the events we want to monitor.</li> 
 <li><strong>Monitoring and Visualization:</strong> Provides dashboards and queries for operational insights.</li> 
</ol> 
<h2>How It Works</h2> 
<p>The solution follows this event flow:</p> 
<ol> 
 <li><strong>Event Generation:</strong> AWS CloudFormation operations in target accounts generate events (stack creation, updates, deletions, resource changes).</li> 
 <li><strong>Event Capture:</strong> <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html">Amazon EventBridge</a> rules in each target account capture these AWS CloudFormation events based on defined patterns.</li> 
 <li><strong>Cross-Account Forwarding:</strong> Events are forwarded to a custom event bus in the management account using cross-account permissions.</li> 
 <li><strong>Centralized Logging:</strong> The central event bus routes all events to a <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html">Amazon CloudWatch Log Group</a> with structured logging.</li> 
 <li><strong>Monitoring and Alerting:</strong> Administrators can view consolidated logs, create custom queries, and set up alerts from a single location.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>Before implementing this solution, ensure you have the following prerequisites in place:</p> 
<ul> 
 <li><strong>AWS account</strong>: Ensure you have valid AWS account.</li> 
 <li><strong>AWS Organizations:</strong> You must have an <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html">AWS Organization</a> structure set up with a primary management account and several member accounts under the management account.</li> 
 <li><strong>Trusted Access:</strong> <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-activate-trusted-access.html">Enable trusted access for AWS CloudFormation StackSets</a> from the management account (this allows StackSets to assume roles in member accounts).</li> 
 <li><strong>Appropriate Permissions:</strong> You must have access to the management account or be configured as a delegated administrator to create and manage StackSets. For detailed information about permissions and security considerations when using StackSets with AWS Organizations, please review the Prerequisites in the AWS CloudFormation StackSets documentation.</li> 
</ul> 
<h2>Implementation Deep Dive</h2> 
<p>The solution is implemented using two AWS CloudFormation templates that work together to create a comprehensive monitoring system:</p> 
<h3>1. Management Account Logging Setup (<a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/CloudFormation/StackSets/templates/log-setup-management.yaml">log-setup-management.yaml</a>)</h3> 
<p>This template establishes the central logging infrastructure in the management account by creating a custom Amazon EventBridge event bus with cross-account access policies and an encrypted Amazon CloudWatch Log Group using a customer-managed&nbsp;<a href="https://quip-amazon.com/arSyA5ZUp7y5/Dev-Platform-Mantler-Project-Candidates">AWS Key Management Service</a> (AWS KMS) key. A key feature is the included stack set resource that automatically deploys the target account configuration to all member accounts, eliminating manual setup and ensuring consistent configuration across the entire organization.</p> 
<h3>2. Stack set Deployment Template (<a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/CloudFormation/StackSets/templates/common-resources-stackset.yaml">common-resources-stackset.yaml</a>)</h3> 
<p>This template creates a service-managed stack set that deploys common resources to all accounts in specified organizational units. The StackSet is configured with auto-deployment enabled to automatically provision new accounts added to the organization and includes operation preferences for parallel regional deployment with fault tolerance settings.</p> 
<h2>Step-by-Step Deployment Guide</h2> 
<h3>Step 1: Download the templates:</h3> 
<ul> 
 <li><a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/CloudFormation/StackSets/templates/log-setup-management.yaml">log-setup-management.yaml</a></li> 
 <li><a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/blob/main/CloudFormation/StackSets/templates/common-resources-stackset.yaml">common-resources-stackset.yaml</a></li> 
</ul> 
<h3>Step 2: Deploy the Management Account Infrastructure</h3> 
<p>Deploy the centralized logging infrastructure to your management account.</p> 
<p>Using <a href="https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html">CLI</a>:</p> 
<p><code>aws cloudformation deploy \</code><br /> <code>&nbsp;&nbsp;--template-file log-setup-management.yaml \</code><br /> <code>&nbsp;&nbsp;--stack-name log-setup-management \</code><br /> <code>&nbsp;&nbsp;--parameter-overrides \</code><br /> <code>&nbsp;&nbsp; &nbsp;OUID=your-organizational-unit-id \</code><br /> <code>&nbsp;&nbsp; &nbsp;OrgID=your-organization-id \</code><br /> <code>&nbsp;&nbsp;--capabilities CAPABILITY_IAM \</code><br /> <code>&nbsp;&nbsp;--region us-east-1</code></p> 
<p><strong>AWS CLI command execution for stack deployment</strong></p> 
<p>Using <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html">AWS Console</a>:</p> 
<ol> 
 <li>Open the AWS CloudFormation console at <a href="https://console.aws.amazon.com/cloudformation">https://console.aws.amazon.com/cloudformation</a>.</li> 
 <li>On the Stacks page, choose <strong>Create</strong> stack at top right, and then choose <strong>With new resources (standard)</strong>.</li> 
 <li>On the Create stack page, <strong>Upload a template file,</strong> choose <strong>Choose File</strong> to choose a template file from your local computer.</li> 
 <li>Choose <strong>Next</strong> to continue and to validate the template.</li> 
 <li>On the Specify stack details page, type a stack name in the Stack name box.</li> 
 <li>In the Parameters section, specify values for the parameters that were defined in the template.</li> 
 <li>Choose <strong>Next</strong> to continue creating the stack.</li> 
 <li><strong>Acknowledge capabilities and transforms</strong>.</li> 
 <li>Choose <strong>Next</strong> to continue.</li> 
 <li>Choose <strong>Submit</strong> to launch your stack.</li> 
</ol> 
<p>This single deployment:</p> 
<ol> 
 <li>Creates the central logging infrastructure in the management account.</li> 
 <li>Automatically deploys Amazon EventBridge rules to all accounts in the specified OU.</li> 
 <li>Sets up the necessary IAM roles and policies for cross-account access.</li> 
</ol> 
<p><img alt="Figure 2:&nbsp;Screenshot showing successful deployment of log-setup-management.yaml template in the management account" class="aligncenter size-full wp-image-24112" height="579" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/22/Screenshot-showing-successful-deployment-of-common-resources-stackset-2.png" width="1456" /></p> 
<p><strong>Figure 2.1: Screenshot showing successful deployment of log-setup-management.yaml template in the management account</strong></p> 
<p><img alt="Figure 2.2:&nbsp;Screenshot showing deployment timeline of log-setup-management.yaml template in the management account" class="aligncenter size-full wp-image-24115" height="390" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/22/Timeline-view-1.png" width="883" /></p> 
<p><strong>Figure 2.2: Deployment timeline view of log-setup-management.yaml template in the management account</strong></p> 
<h3>Step 3: Deploy Common Resources</h3> 
<p>Deploy the sample common resources to demonstrate the logging functionality.</p> 
<p>Using <a href="https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html">CLI</a>:</p> 
<p><code>aws cloudformation deploy \</code><br /> <code>&nbsp;&nbsp;--template-file common-resources-stackset.yaml \</code><br /> <code>&nbsp;&nbsp;--stack-name common-resources-stackset \</code><br /> <code>&nbsp;&nbsp;--parameter-overrides \</code><br /> <code>&nbsp;&nbsp; &nbsp;OUID=your-organizational-unit-id \</code><br /> <code>&nbsp;&nbsp;--capabilities CAPABILITY_IAM \</code><br /> <code>&nbsp;&nbsp;--region us-east-1</code></p> 
<p><strong>AWS CLI command execution for stack deployment</strong></p> 
<p>Using <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html">AWS Console</a>:</p> 
<ol> 
 <li>Open the AWS CloudFormation console at <a href="https://console.aws.amazon.com/cloudformation">https://console.aws.amazon.com/cloudformation</a>.</li> 
 <li>On the Stacks page, choose <strong>Create</strong> stack at top right, and then choose <strong>With new resources (standard)</strong>.</li> 
 <li>On the Create stack page, <strong>Upload a template file</strong>, choose <strong>Choose File</strong> to choose a template file from your local computer.</li> 
 <li>Choose <strong>Next</strong> to continue and to validate the template.</li> 
 <li>On the Specify stack details page, type a stack name in the Stack name box.</li> 
 <li>In the Parameters section, specify values for the parameters that were defined in the template.</li> 
 <li>Choose <strong>Next</strong> to continue creating the stack.</li> 
 <li><strong>Acknowledge capabilities and transforms.</strong></li> 
 <li>Choose <strong>Next</strong> to continue.</li> 
 <li>Choose <strong>Submit</strong> to launch your stack.</li> 
</ol> 
<p>This creates a stack set that deploys&nbsp;Amazon Simple Storage Service (Amazon S3) infrastructure to all target accounts, generating AWS CloudFormation events that will be captured by your centralized logging system.</p> 
<p><img alt="Screenshot showing successful deployment of common-resources-stackset.yaml template for target accounts" class="aligncenter size-full wp-image-24111" height="455" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/22/common-stack-set.png" width="1458" /></p> 
<p><strong>Figure 3: Screenshot showing successful deployment of common-resources-stackset.yaml template for target accounts</strong></p> 
<h3>Step 4: Validation and Testing</h3> 
<p>Confirm event flow and monitoring functionality by viewing the log streams in the ‘central-cloudformation-logs’ log group.</p> 
<h3>Monitoring and Visualization</h3> 
<p>The centralized logging solution provides advanced monitoring capabilities through Amazon CloudWatch Logs Insights and custom dashboards.</p> 
<p>You can customize your queries to get:</p> 
<ul> 
 <li>Recent AWS CloudFormation events across all accounts.</li> 
 <li>Failed stack operations for quick troubleshooting.</li> 
 <li>Successful deployments for verification.</li> 
 <li>Event distribution by account and region.</li> 
 <li>Status breakdown of all AWS CloudFormation operations.</li> 
</ul> 
<p>The following query helps you analyze CloudFormation events across your organization by showing:</p> 
<ul> 
 <li>Timestamp of events</li> 
 <li>Account ID where the event occurred</li> 
 <li>Region of deployment</li> 
 <li>Resource types being deployed</li> 
 <li>Deployment status</li> 
 <li>Logical resource identifiers</li> 
</ul> 
<p><code>fields&nbsp;@timestamp, account, region</code><br /> <code>| parse&nbsp;@message&nbsp;/"resource-type":"(?&lt;resource_type&gt;[^"]+)"/&nbsp;</code><br /> <code>| parse&nbsp;@message&nbsp;/"status":"(?&lt;status&gt;[^"]+)"/&nbsp;</code><br /> <code>| parse&nbsp;@message&nbsp;/"logical-resource-id":"(?&lt;logical_resource_id&gt;[^"]+)"/&nbsp;</code><br /> <code>| sort&nbsp;@timestamp&nbsp;desc</code></p> 
<p><img alt="Figure 4: CloudWatch Logs Insights query results showing CloudFormation events across accounts" class="aligncenter size-full wp-image-24110" height="596" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/22/CloudWatch-Logs-Insights-2.png" width="1246" /></p> 
<p><strong>Figure 4: CloudWatch Logs Insights query results showing CloudFormation events across accounts</strong></p> 
<p>You can customize your queries to filter for specific conditions such as failed deployment status, particular resource types, or specific accounts to quickly identify and troubleshoot issues across your organization’s AWS CloudFormation deployments.</p> 
<h3>Cost Implications</h3> 
<p>When implementing this centralized monitoring solution, you should consider the following cost components:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/eventbridge/pricing/"><strong>Amazon EventBridge pricing</strong></a> – Costs associated with events being published across accounts to the central event bus</li> 
 <li><a href="https://aws.amazon.com/cloudwatch/pricing/"><strong>Amazon CloudWatch pricing</strong></a> – Storage costs for the centralized log group storing CloudFormation events from all accounts. Query costs when analyzing the centralized logs</li> 
 <li><a href="https://aws.amazon.com/kms/pricing/"><strong>AWS Key Management Service pricing</strong></a> – Costs related to the customer-managed key used for log encryption</li> 
</ul> 
<h2>Clean up</h2> 
<p>To clean up the resources created in this solution, follow these steps:</p> 
<ol> 
 <li>First, delete the common resources stack set (common-resources-stackset) from the AWS CloudFormation console in your management account. This will remove all the resources deployed across your member accounts.</li> 
 <li>After the stack set operations are complete, delete the management account logging setup stack (log-setup-management) to remove the centralized logging infrastructure, including the event bus, log groups, and associated IAM roles.</li> 
</ol> 
<p><strong>Note</strong>: Make sure all stack set operations are complete before deleting the management account logging setup to ensure proper cleanup of all resources.</p> 
<h2>Conclusion</h2> 
<p>Managing infrastructure across multiple AWS accounts doesn’t have to be complex. By centralizing AWS CloudFormation logs, you can gain visibility into your multi-account deployments, troubleshoot issues more efficiently, and&nbsp;help achieve consistent resource deployment across your organization.</p> 
<p>This solution demonstrates how AWS services like AWS CloudFormation StackSets, Amazon EventBridge, and Amazon CloudWatch Logs can be combined to create a powerful monitoring system for your infrastructure as code deployments.</p> 
<p>Get started today by implementing this solution in your AWS Organization to gain immediate visibility into your multi-account deployments. Download the templates from our <a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/tree/main/CloudFormation/StackSets/templates">GitHub repository</a> and follow the step-by-step guide to enhance your cloud operations.</p> 
<h2>Authors:</h2> 
<footer> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">Fatima Bzioui</h3> 
  <p style="text-align: left;">Fatima Bzioui is a Cloud Support Engineer with a focus on DevOps best practices and cloud-native solutions. Fatima’s expertise includes Infrastructure as Code and CI/CD implementations, which she uses to help organizations overcome complex technical challenges and achieve their cloud goals.</p> 
 </div> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">Idriss Laouali Abdou</h3> 
  <p style="text-align: left;">Idriss Laouali Abdou is a Sr. Product Manager Technical for AWS Infrastructure-as-Code based in Seattle. He focuses on improving developer productivity through StackSets and CloudFormation Infrastructure provisioning experiences. Outside of work, you can find him creating educational content for thousands of students, cooking, or dancing.</p> 
 </div> 
</footer>
