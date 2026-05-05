---
title: "Introducing the AWS Infrastructure as Code MCP Server: AI-Powered CDK and CloudFormation Assistance"
url: "https://aws.amazon.com/blogs/devops/introducing-the-aws-infrastructure-as-code-mcp-server-ai-powered-cdk-and-cloudformation-assistance/"
date: "Fri, 28 Nov 2025 22:52:07 +0000"
author: "Idriss Laouali Abdou"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<p>Streamline your AWS infrastructure development with AI-powered documentation search, validation, and troubleshooting</p> 
<h1>Introduction</h1> 
<p>Today, we’re excited to introduce the <a href="https://awslabs.github.io/mcp/servers/aws-iac-mcp-server">AWS Infrastructure-as-Code (IaC) MCP Server</a>, a new tool that bridges the gap between AI assistants and your AWS infrastructure development workflow. Built on the Model Context Protocol (MCP), this server enables AI assistants like <a href="https://kiro.dev/cli/">Kiro CLI</a>, Claude or Cursor to help you search <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> and&nbsp;<a href="https://aws.amazon.com/cdk/">Cloud Development Kit (CDK)</a> documentation, validate templates, troubleshoot deployments, and follow best practices – all while maintaining the security of local execution.</p> 
<p>Whether you’re writing AWS CloudFormation templates or AWS Cloud Development Kit (CDK) code, the IaC MCP Server acts as an intelligent companion that understands your infrastructure needs and provides contextual assistance throughout your development lifecycle.</p> 
<p>The&nbsp;<a href="https://modelcontextprotocol.io/">Model Context Protocol (MCP)</a>&nbsp;is an open standard that enables AI assistants to securely connect to external data sources and tools. Think of it as a universal adapter that lets AI models interact with your development tools while keeping sensitive operations local and under your control.</p> 
<p>The IaC MCP Server provides nine specialized tools organized into two categories:</p> 
<h2><strong>Remote Documentation Search Tools</strong></h2> 
<p>These tools connect to the AWS Knowledge MCP backend to retrieve relevant, up-to-date information:</p> 
<ol> 
 <li><strong>&nbsp;search_cdk_documentation</strong><br /> Search the AWS CDK knowledge base for APIs, concepts, and implementation guidance.</li> 
 <li><strong>search_cdk_samples_and_constructs</strong><br /> Discover pre-built AWS CDK constructs and patterns from the AWS Construct Library.</li> 
 <li><strong>search_cloudformation_documentation</strong><br /> Query CloudFormation documentation for resource types, properties, and intrinsic functions.</li> 
 <li><strong>read_iac_documentation_page</strong><br /> Retrieve and read full CloudFormation and CDK documentation pages returned from searches or provided URLs.</li> 
</ol> 
<h2><strong>Local Validation and Troubleshooting Tools</strong></h2> 
<p>These tools run entirely on your machine</p> 
<ol> 
 <li><strong>cdk_best_practices</strong><br /> Access a curated collection of AWS CDK best practices and design principles.</li> 
 <li><strong>validate_cloudformation_template</strong><br /> Perform syntax and schema validation using&nbsp;cfn-lint&nbsp;to catch errors before deployment.</li> 
 <li><strong>check_cloudformation_template_compliance</strong><br /> Run security and compliance checks against your templates using AWS Guard rules and&nbsp;cfn-guard.</li> 
 <li><strong>troubleshoot_cloudformation_deployment</strong><br /> Analyze CloudFormation stack deployment failures with integrated CloudTrail event analysis. This tool will use your AWS credentials to analyze your stack status.</li> 
 <li><strong>get_cloudformation_pre_deploy_validation_instructions<br /> </strong>Returns instructions for CloudFormation’s pre-deployment validation feature, which validates templates during change set creation.</li> 
</ol> 
<h3><strong>Key Use Cases</strong></h3> 
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
<h1>Architecture and Security</h1> 
<h2>Security Design</h2> 
<p><strong>Local Execution:</strong> The MCP server runs entirely on your local machine using uv (the fast Python package manager). No code or templates are sent to external services except for documentation searches.</p> 
<p><strong>AWS Credentials:</strong> The server uses your existing AWS credentials (from&nbsp;~/.aws/credentials, environment variables, or IAM roles) to access CloudFormation and CloudTrail APIs. This follows the same security model as the AWS CLI.</p> 
<p><strong>stdio Communication:</strong> The server communicates with AI assistants over standard input/output (stdio), with no network ports opened.</p> 
<p><strong>Minimal Permissions:</strong> For full functionality, the server requires read-only access to CloudFormation stacks and CloudTrail events—no write permissions needed for validation and troubleshooting workflows.</p> 
<h1>Getting Started</h1> 
<h2>Prerequisites</h2> 
<ul> 
 <li>Python 3.10 or later<br /> uv&nbsp;package manager<br /> AWS credentials configured locally<br /> MCP-compatible AI client (e.g., Kiro CLI, Claude Desktop)</li> 
</ul> 
<h2>Configuration</h2> 
<p>Configure the MCP server in your MCP client configuration. For this blog we will focus on Kiro CLI. Edit&nbsp;.kiro/settings/mcp.json):</p> 
<pre><code class="lang-json">{
  "mcpServers": {
    "awslabs.aws-iac-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.aws-iac-mcp-server@latest"],
      "env": {
        "AWS_PROFILE": "your-named-profile",
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
</code></pre> 
<h2>Security Considerations</h2> 
<p><strong>Privacy Notice</strong>: This MCP server executes AWS API calls using your credentials and shares the response data with your third-party AI model provider (e.g., Amazon Q, Claude Desktop, Cursor, VS Code). Users are responsible for understanding your AI provider’s data handling practices and ensuring compliance with your organization’s security and privacy requirements when using this tool with AWS resources.</p> 
<h3>IAM Permissions</h3> 
<p>The MCP server requires the following AWS permissions:</p> 
<p><strong>For Template Validation and Compliance:</strong></p> 
<ul> 
 <li>No AWS permissions required (local validation only)</li> 
</ul> 
<p><strong>For Deployment Troubleshooting:</strong></p> 
<ul> 
 <li>cloudformation:DescribeStacks</li> 
 <li>cloudformation:DescribeStackEvents</li> 
 <li>cloudformation:DescribeStackResources</li> 
 <li>cloudtrail:LookupEvents (for CloudTrail deep links)</li> 
</ul> 
<p>Example IAM policy:</p> 
<pre><code class="lang-json">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:DescribeStacks",
        "cloudformation:DescribeStackEvents",
        "cloudformation:DescribeStackResources",
        "cloudtrail:LookupEvents"
      ],
      "Resource": "*"
    }
  ]
}
</code></pre> 
<h3>Example Use Case With Kiro CLI</h3> 
<p><strong>IMPORTANT: Ensure you have satisfied all prerequisites before attempting these commands.</strong></p> 
<p>1. With the&nbsp;mcp.json&nbsp;file correctly set, try to run a sample prompt. In your terminal, run kiro-cli chat to start using Kiro-cli in the CLI.</p> 
<p><img alt="Figure 1: Kiro-CLI with AWS IaC MCP server " class="aligncenter size-full wp-image-24608" height="752" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-1-Kiro-CLI-with-AWS-IaC-MCP-server-.png" width="638" /></p> 
<p style="text-align: center;"><strong>Figure 1: Kiro-CLI with AWS IaC MCP server</strong></p> 
<h3>Scenarios:</h3> 
<ul> 
 <li><strong>“What are the CDK best practices for Lambda functions?”</strong></li> 
</ul> 
<p><img alt="Figure 2 Search the CDK best practices for Lambda functions" class="aligncenter size-full wp-image-24611" height="955" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-2-Search-the-CDK-best-practices-for-Lambda-functions.png" width="574" /></p> 
<p style="text-align: center;"><strong>Figure 2: Search the CDK best practices for Lambda functions</strong></p> 
<ul> 
 <li><strong>“Search for CDK samples that use DynamoDB with Lambda”</strong></li> 
</ul> 
<p><img alt="Figure 3: Search for CDK samples that use DynamoDB with Lambda" class="aligncenter size-full wp-image-24612" height="906" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-3-Search-for-CDK-samples-that-use-DynamoDB-with-Lambda.png" width="637" /></p> 
<p style="text-align: center;"><strong>Figure 3: Search for CDK samples that use DynamoDB with Lambda</strong></p> 
<ul> 
 <li><strong>“Validate my CloudFormation template at ./template.yaml”</strong></li> 
</ul> 
<p><img alt="Figure 4: Validate my CloudFormation template with AWS IaC MCP Server" class="aligncenter size-full wp-image-24621" height="972" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Figure-4-Validate-my-CloudFormation-template-with-AWS-IaC-MCP-Server-1.png" width="639" /></p> 
<p style="text-align: center;"><strong>Figure 4: Validate my CloudFormation template with AWS IaC MCP Server</strong></p> 
<ul> 
 <li><strong>“Check if my template complies with security best practices”</strong></li> 
</ul> 
<p><img alt="Figure 5: Check if my template complies with security best practices with AWS IaC MCP Server" class="aligncenter size-full wp-image-24614" height="363" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/28/Screenshot-2025-11-28-at-12.10.01 PM.png" width="637" /></p> 
<p style="text-align: center;"><strong>Figure 5: Check if my template complies with security best practices with AWS IaC MCP Server</strong></p> 
<h2>Best Practices</h2> 
<ul> 
 <li><strong>Start with Documentation Search:</strong> Before writing code, search for existing constructs and patterns</li> 
 <li><strong>Validate Early and Often:</strong> Run validation tools before attempting deployment</li> 
 <li><strong>Check Compliance:</strong> Use check_template_compliance to catch security issues during development</li> 
 <li><strong>Leverage CloudTrail:</strong> When troubleshooting, the CloudTrail integration provides detailed failure context</li> 
 <li><strong>Follow CDK Best Practices:</strong> Use the cdk_best_practices tool to align with AWS recommendations</li> 
</ul> 
<h2>What’s Next?</h2> 
<p>The IAC MCP Server represents a new paradigm in the AI agentic workflow infrastructure development – one where AI assistants understand your tools, help you navigate complex documentation, and provide intelligent assistance throughout the development lifecycle.</p> 
<h2>Get Involved</h2> 
<p>The AWS IaC MCP Server is available now:</p> 
<ul> 
 <li><strong>Documentation and GitHub Repository:</strong> <a href="https://awslabs.github.io/mcp/servers/aws-iac-mcp-server">aws-iac-mcp-server</a></li> 
 <li><strong>Feedback:</strong> We welcome issues and pull requests! Or respond to our IaC survey here.</li> 
</ul> 
<p>Ready to supercharge your infrastructure as code development? Install the IaC MCP Server today and experience AI-powered assistance for your AWS CDK and CloudFormation workflows.</p> 
<p>Have questions or feedback? Reach out to the blog authors on the AWS Developer Forums.</p> 
<h2><strong>About Authors</strong></h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="127" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/idriss-profile-cut-scaled.jpg" width="127" />
  </div> 
  <h3 class="lb-h4">Idriss Laouali Abdou</h3> 
  <p style="text-align: left;">Idriss is a Sr. Product Manager Technical on the AWS Infrastructure-as-Code team based in Seattle. He focuses on improving developer productivity through AWS CloudFormation and StackSets Infrastructure provisioning experiences. Outside of work, you can find him creating educational content for thousands of students, cooking, or dancing.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/08/09/brian-terry.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Brian Terry</h3> 
  <p style="text-align: left;">Brian Terry, Senior WW Data &amp; AI PSA, is an innovation leader with more than 20 years of experience in technology and engineering. Brian is pursuing a PhD in computer science at the University of North Dakota and has spearheaded generative AI projects, optimized infrastructure scalability, and driven partner integration strategies. He is passionate about leveraging technology to deliver scalable, resilient solutions that foster business growth and innovation.</p> 
 </div> 
</footer>
