---
title: "AWS CloudFormation 2025 Year In Review"
url: "https://aws.amazon.com/blogs/devops/aws-cloudformation-2025-year-in-review/"
date: "Wed, 28 Jan 2026 01:08:08 +0000"
author: "Idriss Laouali Abdou"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<p><a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> enables you to model and provision your cloud application infrastructure as code-base templates. Whether you prefer writing templates directly in JSON or YAML, or using programming languages like Python, Java, and TypeScript with the <a href="https://aws.amazon.com/cdk/">AWS Cloud Development Kit (CDK)</a>, CloudFormation and CDK provide the flexibility you need. For organizations adopting <a href="https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html">multi-account strategies</a>, CloudFormation <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html">StackSets</a> offers a powerful capability to deploy resources across multiple regions and accounts in parallel.</p> 
<p>In 2025, we delivered a comprehensive set of major enhancements focused on three core areas: reducing dev-test cycle through early validation, improving deployment safety with improved configuration drift management, and integrating IaC context to AI-powered development tools.</p> 
<p>These launches address common pain points in infrastructure development workflows, from catching deployment errors before resource provisioning to managing configuration drift systematically. The features span the entire development lifecycle, from template authoring in your IDE to multi-account deployments at scale.</p> 
<p>This blog provides an overview of the key capabilities we launched in 2025 and how they improve your infrastructure development workflow.</p> 
<h1>Accelerating Development Cycles</h1> 
<h2>Early Validation &amp; Enhanced Troubleshooting: Pre-Deployment Error Detection</h2> 
<p>CloudFormation now validates your templates during <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html">change set</a> (preview of infrastructure changes before deployment) creation, catching common deployment errors before resource provisioning begins. The validation checks for invalid property syntax, resource name conflicts with existing resources in your account, and S3 bucket emptiness constraints on delete operations.</p> 
<p><img alt="Figure 1: Pre-deployment validations view" class="aligncenter size-full wp-image-24249" height="765" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/CloudFormation-Pre-Deployment-validation-view.png" width="1170" /></p> 
<p style="text-align: center;"><strong>Figure 1: Pre-deployment validations view</strong></p> 
<p>When validation fails, the change set status shows ‘FAILED’ with detailed information about each issue, including the property path where problems occur. This early feedback helps you fix issues faster rather than waiting for deployment failures.</p> 
<p style="text-align: center;"><img alt="Figure 2: CloudFormation Validation of Invalid ENUM value for nested property" class="aligncenter size-full wp-image-24282" height="1850" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/CloudFormation-Validation-of-Invalid-ENUM-value-for-nested-property.png" width="2440" /><br /> <strong>Figure 2: Validation of Invalid ENUM value for nested property</strong></p> 
<h2>Improved Deployment troubleshooting</h2> 
<p>For runtime errors that occur during deployment, every stack operation now receives a unique operation ID. You can filter stack events by operation ID to quickly identify root causes, reducing troubleshooting time from minutes to seconds. The new <a href="https://docs.aws.amazon.com/cli/latest/reference/cloudformation/describe-events.html">describe-events</a> API provides grouped access to events. You can query events for a specific operation, filter to FAILED status events, and extract the root cause without parsing through the entire stack event history.</p> 
<p style="text-align: center;"><img alt="Figure 3: New CloudFormation stack operation page" class="aligncenter size-full wp-image-24289" height="1758" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/CloudFormation-Stack-Info-page-showing-new-operation-IDs-4.png" width="3424" /><strong>Figure 3: New CloudFormation stack operation page</strong></p> 
<p style="text-align: center;"><img alt="Figure 4: Filter operation failure root causes" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/CloudFormation-Filter-operation-failure-root-causes.png" width="3378" /><br /> <strong>Figure 4: Filter operation failure root causes</strong></p> 
<p><strong>Learn more</strong>:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/about-aws/whats-new/2025/11/cloudformation-dev-test-cycle-validation-troubleshooting/">What’s New Post</a></li> 
 <li><a href="https://aws.amazon.com/blogs/devops/accelerate-infrastructure-development-with-cloudformation-pre-deployment-validation-and-simplified-troubleshooting/">Detailed Blog Post</a></li> 
</ul> 
<h2>CloudFormation IDE Experience: Language Server Protocol Integration</h2> 
<p>We launched the <a href="https://aws.amazon.com/about-aws/whats-new/2025/11/aws-cloudformation-intelligent-authoring-ides/">AWS CloudFormation Language Server</a>, bringing end-to-end infrastructure development directly into your IDE. Available through the AWS Toolkit for Visual Studio Code, Kiro, and other compatible IDEs, this capability transforms how you author CloudFormation templates.</p> 
<p style="text-align: center;"><img alt="Figure 1: Filter operation failure root causes" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/cfn_init_1-1-1.gif" width="3378" /></p> 
<p style="text-align: center;"><strong>Figure 1: Initializing a CloudFormation project with environment configuration</strong></p> 
<p>The Language Server provides context-aware auto-completion that understands CloudFormation semantics. When you define resources, it suggests only required properties automatically, while optional properties appear on hover. Built-in validation catches issues before deployment integrating early validation capabilities, flagging invalid resource properties, missing IAM permissions, and security policy violations using CloudFormation Guard.</p> 
<p style="text-align: center;"><strong>Figure 2: Hover information displaying optional properties and their documentation</strong></p> 
<p>The drift-aware deployment view highlights differences between your template and deployed infrastructure, helping you spot configuration changes made outside CloudFormation. The Language Server also provides semantic navigation features, go-to-definition for logical IDs, find-all-references for resource dependencies, and hover documentation that pulls from the CloudFormation resource specification. These features work across intrinsic functions like !Ref, !GetAtt, and !Sub, understanding the CloudFormation template structure. By integrating validation and real-time feedback directly into your authoring experience, the Language Server keeps you in flow state, reducing context switching between your IDE, AWS Console, and documentation.</p> 
<p style="text-align: center;"><strong>Figure 3: Type-aware completions for intrinsic functions like !GetAtt &amp; !Ref</strong></p> 
<p><strong>Learn more:</strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/about-aws/whats-new/2025/11/aws-cloudformation-intelligent-authoring-ides/">What’s New Post</a></li> 
 <li><a href="https://aws.amazon.com/blogs/devops/announcing-cloudformation-ide-experience-end-to-end-development-in-your-ide/">Detailed Blog Post</a></li> 
</ul> 
<h3 style="text-align: left;"><strong>Stack Refactoring: Adapt your infrastructure to your organization evolution</strong></h3> 
<p>Stack Refactoring enables you to reorganize your CloudFormation and CDK infrastructure without disrupting deployed resources. You can move resources between stacks, rename logical IDs, and decompose monolithic stacks into focused components while maintaining resource stability and operational state.</p> 
<p>Whether you’re modernizing legacy stacks, aligning infrastructure with evolving architectural patterns, or improving long-term maintainability, Stack Refactoring adapts your CloudFormation and CDK organization to changing requirements. The console and CDK experience, launched this year, extends the earlier CLI capability, making refactoring accessible through your preferred interface.</p> 
<p style="text-align: center;"><img alt="Provide a description to help you identify your stack refactor." class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/21/blogpicture1-1024x423.png" width="3378" /></p> 
<p><strong>Learn more:</strong><br /> – <a href="https://aws.amazon.com/blogs/devops/introducing-aws-cloudformation-stack-refactoring-reorganize-your-infrastructure-without-disruption/">Blog Post – Refactor CloudFormation from Console</a><br /> – <a href="https://aws.amazon.com/blogs/devops/aws-cloud-development-kit-cdk-launches-refactor/">Blog Post – Refactor CDK</a></p> 
<h1>Safer Deployments</h1> 
<h2>Drift-Aware Change Sets</h2> 
<p>Configuration drift occurs when infrastructure managed by CloudFormation is modified through the AWS Console, SDK, or CLI. Drift-aware change sets address this challenge by providing a three-way comparison between your new template, last-deployed template, and actual infrastructure state.</p> 
<p style="text-align: center;"><img alt="Examine the drift-aware change set to see the dangerous memory reduction that would occur" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure5-memory-reduction-1-2-1024x762.png" width="3378" /></p> 
<p style="text-align: center;"><img alt="Examine the drift-aware change set to see the dangerous memory reduction that would occur" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure6-view-drift-1-2-1024x749.png" width="3378" /></p> 
<p style="text-align: center;"><strong>Figure 4: Examine the drift-aware change set to see the dangerous memory reduction that would occur</strong></p> 
<p>This capability helps you prevent unexpected overwrites of drift. If your change set preview shows unintended changes, you can update your template values and recreate the change set before deployment. During execution, CloudFormation matches resource properties with template values and recreates resources deleted outside of CloudFormation.</p> 
<p>Drift-aware change sets enable you to systematically revert drift and keep infrastructure in sync with templates, strengthening reproducibility for testing and disaster recovery while maintaining your security posture.</p> 
<p><strong>Learn more: </strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/about-aws/whats-new/2025/11/configuration-drift-enhanced-cloudformation-sets/">What’s New Post</a></li> 
 <li><a href="https://aws.amazon.com/blogs/devops/safely-handle-configuration-drift-with-cloudformation-drift-aware-change-sets/">Detailed Blog Post</a></li> 
</ul> 
<h1>Enforcing Proactive Controls</h1> 
<h2>CloudFormation Hooks: Control Catalog with Hooks</h2> 
<p>AWS CloudFormation Hooks now supports managed proactive controls, enabling customers to validate resource configurations against AWS best practices without writing custom Hooks logic. Customers can select controls from the AWS Control Tower Controls Catalog and apply them during CloudFormation operations. When using CloudFormation, customers can configure these controls to run in warn mode, allowing teams to test controls without blocking deployments and giving them the flexibility to evaluate control behavior before enforcing policies in production. This significantly reduces setup time, eliminates manual errors, and ensures comprehensive governance coverage across your infrastructure.</p> 
<p><img alt="" class="aligncenter size-full wp-image-24903" height="922" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/01/28/Control-Catalog-with-Hooks.png" width="2152" /></p> 
<p>AWS also introduced a new Hooks Invocation Summary page in the CloudFormation console. This centralized view provides a complete historical record of Hooks activity, showing which controls were invoked, their execution details, and outcomes such as pass, warn, or fail. This simplifies compliance reporting issues faster.</p> 
<p>With this launch, customers can now leverage AWS-managed controls as part of their provisioning workflows, eliminating the overhead of writing and maintaining custom logic. These controls are curated by AWS and aligned with industry best practices, helping teams enforce consistent policies across all environments. The new summary page delivers essential visibility into Hook invocation history, enabling faster issue resolution and streamlined compliance reporting.</p> 
<p><strong>Learn more:</strong></p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/proactive-controls-hooks.html">AWS CloudFormation Proactive Control Hooks&nbsp;</a></li> 
</ul> 
<h1>Scaling Multi-Account Infrastructure</h1> 
<h2>StackSets Deployment Ordering</h2> 
<p style="text-align: center;"><img alt="Figure : Example of a multi-region AWS CloudFormation StackSet architecture with an administrative account and target accounts" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/17/Image-1.png" width="3378" /></p> 
<p>CloudFormation StackSets now supports deployment ordering for auto-deployment mode, enabling you to define the sequence in which stack instances automatically deploy across accounts and regions. This capability coordinates complex multi-stack deployments where foundational infrastructure must be provisioned before dependent application components.</p> 
<p style="text-align: center;"><img alt="Figure : CloudFormation StackSets Console – Auto-deployment options view" class="aligncenter size-full wp-image-24262" height="1206" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/17/image-25.png" width="3378" /></p> 
<p>When creating or updating a StackSet, you can specify up to 10 dependencies per stack instance using the DependsOn parameter in the AutoDeployment configuration. StackSets automatically orchestrates deployments based on your defined relationships. For example, you can ensure networking and security stack instances complete deployment before application stack instances begin, preventing deployment failures due to missing dependencies.</p> 
<p>StackSets includes built-in cycle detection to prevent circular dependencies and provides error messages to help resolve configuration issues. This feature is available at no additional cost in all AWS Regions where CloudFormation StackSets is available.</p> 
<p><strong>Learn more:</strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/about-aws/whats-new/2025/11/configuration-drift-enhanced-cloudformation-sets/">What’s New Post</a></li> 
 <li><a href="https://aws.amazon.com/blogs/devops/safely-handle-configuration-drift-with-cloudformation-drift-aware-change-sets/">Detailed Blog Post</a></li> 
</ul> 
<h1>AI-Powered Infrastructure Development</h1> 
<h2>AWS IaC Server</h2> 
<p>We introduced the <a href="https://awslabs.github.io/mcp/servers/aws-iac-mcp-server">AWS Infrastructure-as-Code (IaC) MCP Server</a>, bridging AI assistants with your AWS infrastructure development workflow. Built on the Model Context Protocol (MCP), this server enables AI assistants like <a href="https://kiro.dev/cli/">Kiro CLI </a>to help you search CloudFormation and CDK documentation, validate templates, troubleshoot deployments, and follow best practices, all while maintaining the security of local execution.</p> 
<p><img alt="Figure 1: Kiro-CLI with AWS IaC MCP server " class="aligncenter size-full wp-image-24608" height="752" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-1-Kiro-CLI-with-AWS-IaC-MCP-server-.png" width="638" /></p> 
<p style="text-align: center;"><strong>Figure 1: Kiro-CLI with AWS IaC MCP server</strong></p> 
<p>The IaC MCP Server provides nine specialized tools organized into two categories. Remote documentation search tools connect to AWS knowledge bases to retrieve up-to-date information about CloudFormation resources, CDK APIs, and implementation guidance. Local validation and troubleshooting tools run entirely on your machine, performing syntax validation with cfn-lint, security checks with CloudFormation Guard, and deployment failure analysis with integrated CloudTrail events.</p> 
<p><img alt="Figure 4: Validate my CloudFormation template with AWS IaC MCP Server" class="aligncenter size-full wp-image-24621" height="972" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-4-Validate-my-CloudFormation-template-with-AWS-IaC-MCP-Server-1.png" width="639" /></p> 
<p style="text-align: center;"><strong>Figure 4: Validate my CloudFormation template with AWS IaC MCP Server</strong></p> 
<h2>Key Use Cases</h2> 
<ol> 
 <li><strong>Intelligent Documentation Assistant</strong></li> 
</ol> 
<p>Instead of manually searching through documentation, ask your AI assistant natural language questions:</p> 
<p>“How do I create an S3 bucket with encryption enabled in CDK?”</p> 
<p>The server searches CDK best practice and samples, returning relevant code examples and explanations.</p> 
<p><strong>&nbsp; &nbsp; &nbsp;2. Proactive Template Validation</strong></p> 
<p>Before deploying infrastructure changes:</p> 
<p>User: “Validate my CloudFormation template and check for security issues”</p> 
<p>AI Agent: [Uses validate_cloudformation_template and check_cloudformation_template_compliance]</p> 
<p>“Found 2 issues: Missing encryption on EBS volumes,</p> 
<p>and S3 bucket lacks public access block configuration”</p> 
<p><strong> &nbsp;3. Rapid Deployment Troubleshooting</strong></p> 
<p>When a stack deployment fails:</p> 
<p>User: “My stack ‘stack_03’ in us-east-1 failed to deploy. What happened?”</p> 
<p>AI Agent: [Uses troubleshoot_stack_deployment with CloudTrail integration]</p> 
<p>“The deployment failed due to insufficient IAM permissions.</p> 
<p>CloudTrail shows AccessDenied for ec2:CreateVpc.</p> 
<p>You need to add VPC permissions to your deployment role.”</p> 
<p><strong>&nbsp; &nbsp; &nbsp;4. Learning and Exploration</strong></p> 
<p>New to AWS CDK? The server helps you discover constructs and patterns:</p> 
<p>User: “Show me how to build a serverless API”</p> 
<p>AI Agent: [Searches CDK constructs and samples]</p> 
<p>“Here are three approaches using API Gateway + Lambda…”</p> 
<p>Learn more: <a href="https://aws.amazon.com/blogs/devops/safely-handle-configuration-drift-with-cloudformation-drift-aware-change-sets/">Detailed Blog Post</a></p> 
<h1>Learn more</h1> 
<p>Here are some resources to help you get started learning and using CloudFormation to manage your cloud infrastructure:</p> 
<ul> 
 <li><a href="https://youtu.be/_4hvWns9ICY?si=WELIHRgUpdgvuM9P">Watch our re:Invent 2025 session on CloudFormation and CDK</a></li> 
 <li><a href="https://catalog.workshops.aws/cfn101/en-US">AWS CloudFormation Workshop</a></li> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html">AWS CloudFormation Troubleshooting Guide</a></li> 
</ul> 
<h1>Conclusion</h1> 
<p>As we begin 2026, our focus remains on making infrastructure deployment faster, safer, and more manageable. The launches in 2025 reflect our commitment to solving real customer challenges and improving the CloudFormation developer experience. From intelligent IDE integrations to AI-powered assistance, these capabilities help you build infrastructure with greater confidence and efficiency.</p> 
<p>We encourage you to try these features and share your feedback. For detailed information about any of these launches, visit our <a href="https://docs.aws.amazon.com/cloudformation/">documentation</a> or check out the <a href="https://aws.amazon.com/blogs/devops/">AWS DevOps Blog</a>.</p> 
<h2>Blog Author Bio:</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="127" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/idriss-profile-cut-scaled.jpg" width="127" />
  </div> 
  <h3 class="lb-h4">Idriss Laouali Abdou</h3> 
  <p style="text-align: left;">Idriss is a Sr. Product Manager Technical on the AWS Infrastructure-as-Code team based in Seattle. He focuses on improving developer productivity through AWS CloudFormation and StackSets Infrastructure provisioning experiences. Outside of work, you can find him creating educational content for thousands of students, cooking, or dancing.</p> 
 </div> 
</footer>
