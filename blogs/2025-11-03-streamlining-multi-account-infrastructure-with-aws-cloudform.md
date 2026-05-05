---
title: "Streamlining Multi-Account Infrastructure with AWS CloudFormation StackSets and AWS CDK"
url: "https://aws.amazon.com/blogs/devops/streamlining-multi-account-infrastructure-with-aws-cloudformation-stacksets-and-aws-cdk/"
date: "Mon, 03 Nov 2025 14:44:41 +0000"
author: "Franco Abregu"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<h2>Introduction</h2> 
<p>Organizations operating at scale on AWS often need to manage resources across multiple accounts and regions. Whether it’s deploying security controls, compliance configurations, or shared services, maintaining consistency can be challenging.</p> 
<p><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html">AWS CloudFormation StackSets</a> (StackSets) has been helping organizations deploy resources across multiple accounts and regions since its launch. While the service is powerful on its own, combining it with Infrastructure as Code (IaC) tools and implementing automated deployments can significantly enhance its capabilities.</p> 
<p>In this post, we’ll show you how to leverage AWS CloudFormation StackSets at scale using <a href="https://docs.aws.amazon.com/cdk/v2/guide/home.html">AWS CDK</a> and implement a robust CI/CD pipeline for automated deployments with <a href="https://aws.amazon.com/codepipeline/">AWS CodePipeline</a>.</p> 
<h2>StackSets key concepts</h2> 
<p><strong>AWS CloudFormation StackSets</strong> allows you to create, update, or delete CloudFormation stacks across multiple AWS accounts and regions with a single operation. It’s essentially a way to manage infrastructure at scale across your AWS organization. Using an administrator account, you define and manage a CloudFormation template, and use the template as the basis for provisioning stacks into selected target accounts across specified AWS Regions:</p> 
<p><img alt="StackSets Overview" class="aligncenter wp-image-24153 size-full" height="491" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/03/StackSets-Page-1.drawio.png" width="871" /></p> 
<p style="text-align: center;"><strong>Figure 1. StackSets overview.</strong></p> 
<p>The <strong>Administrator Account</strong> is the AWS account where you create and manage StackSets and the <strong>Target Accounts</strong> are the AWS accounts where the stack instances are deployed.</p> 
<p>The <strong>Stack Instances </strong>are individual stacks created from the StackSet template deployed to specific account-region combinations.</p> 
<p><strong>&nbsp;</strong>You can make the following operations using StackSets: Create, update, and delete actions performed on stack instances. These operations can be applied in concurrent or sequential way.</p> 
<p>Sequential Deployment:</p> 
<ul> 
 <li>Account-by-account deployment</li> 
 <li>Region-by-region within accounts</li> 
 <li>Configurable failure thresholds</li> 
</ul> 
<p>Parallel Deployment:</p> 
<ul> 
 <li>Concurrent account deployments</li> 
 <li>Maximum concurrent account setting</li> 
 <li>Region priority configuration</li> 
</ul> 
<p>Hybrid Deployment:</p> 
<ul> 
 <li>Combine sequential and parallel</li> 
 <li>Account group-based deployment</li> 
 <li>Regional deployment strategies</li> 
</ul> 
<h2>The power of StackSets</h2> 
<p>The use of StackSets allows us to extend AWS CloudFormation’s capabilities in several important ways:</p> 
<h3>Governance</h3> 
<p>It provides you with <strong>Centralized Management</strong> as a single point of control while including consistent deployment patterns and automated stack instance management across AWS accounts and regions.</p> 
<p>With <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-drift.html">Drift Detection</a> feature, you can identify if any of the stack instances of your StackSet have configuration differences according to its expected configuration. You detect changes made outside CloudFormation and changes made to an instance stack through CloudFormation directly without using the StackSet.</p> 
<h3>Flexible Deployment</h3> 
<p>You also have flexible deployment options with controlled rollout. For example, with <strong>Concurrent Deployments</strong> you can deploy to multiple accounts within each region simultaneously while controlling deployment order. It also includes failure tolerance with automated retry failed operations.</p> 
<h3>Operational Efficiency</h3> 
<p>It reduces manual effort in managing multi-account and multi-region environments while minimizes human error in deployments.</p> 
<h3>Cost Management</h3> 
<p>It delivers comprehensive resource organization and streamlined tracking of resources across accounts and regions containing instance stacks. Using centralized management, simplifies the resource tracking and organization enabling you you to have:</p> 
<ul> 
 <li><strong>unified visibility: </strong>view all related stacks from a single StackSet console (with their deployment status)</li> 
 <li><strong>consistent tagging: </strong>apply standardized tags across all stack instances for cost allocation and resource grouping</li> 
 <li><strong>drift detection:</strong> run drift detection across all stack instances simultaneously</li> 
 <li><strong>operations tracking:</strong> track all operations (create, update and delete) across account/regions from one place</li> 
</ul> 
<h3>Built-in Safety</h3> 
<p>You can establish maximum concurrent operation limits, failure tolerance thresholds and automatic retry mechanisms. You also have recovery capabilities through update operations. All these features make a built-in safety mechanisms that prevent widespread failures.</p> 
<p>Let’s say you have 100 target accounts, with the maximum concurrent limits, you can for example deploy a change to only 10 accounts. Also, with a failure threshold you can set how many failures do you allow before automatically stopping the process (e.g., stop if more than 5 accounts fail). This way you can gradually deploy and test your templates with a little group, establishing failure thresholds, instead of affecting the stacks preventing&nbsp;mass failures.</p> 
<p>When an operation fails, AWS CloudFormation performs a rollback in the stack instances deploying the previous working template. You will still need to correct the template and apply it again in all the stack instances. With StackSets, you can fix the issues in the template and run again an update across all the stacks including the concurrent limit and failure threshold mentioned before to safety test the fix.</p> 
<h3>Security and Compliance management</h3> 
<p>This security-focused approach with StackSets helps organizations maintain a strong security posture across their AWS environment while reducing the operational overhead of managing security at scale.</p> 
<p>You can use StackSets to deploy standardized security policies across accounts, enforce security baselines automatically and implement security guardrails organization-wide. For example, you can deploy detective control resource and its configuration in all your accounts like Amazon GuardDuty or Amazon Macie. You can also deploy preventive controls like SCPs, AWS Firewall Manager or AWS Shield Advanced. For example you can deploy through StackSets the following CloudFormation template en each target account to block certain actions in a region:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-yaml">&lt;code&gt;AWSTemplateFormatVersion: '2010-09-09'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;Description: 'Service Control Policy to block access to specific AWS regions'&lt;/code&gt;&lt;br /&gt;&lt;br /&gt;&lt;code&gt;Parameters:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;PolicyName:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Type: String&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Default: 'RegionDenyPolicy'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'Name for the Service Control Policy'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;PolicyDescription:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Type: String&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Default: 'Blocks access to Singapore region (ap-southeast-1) while allowing global services'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'Description for the Service Control Policy'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;BlockedRegion:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Type: String&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Default: 'ap-southeast-1'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'AWS Region to block access to'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;AllowedValues:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;- 'ap-southeast-1'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;- 'ap-southeast-2'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;- 'eu-west-3'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;- 'us-west-1'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;- 'ca-central-1'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;TargetOUId:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Type: String&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'Organizational Unit ID to attach the policy to (e.g., ou-root-xxxxxxxxxx)'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;&lt;/code&gt;&lt;br /&gt;&lt;code&gt;Resources:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;RegionDenySCP:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Type: AWS::Organizations::Policy&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Properties:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Name: !Ref PolicyName&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Description: !Ref PolicyDescription&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Type: SERVICE_CONTROL_POLICY&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Content:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;Version: '2012-10-17'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;Statement:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- Sid: DenyAccessToSpecificRegion&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Effect: Deny&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;NotAction:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- 'route53:*'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- 'cloudfront:*'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- 'sts:*'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Resource: '*'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Condition:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;StringEquals:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'aws:RequestedRegion':&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- !Ref BlockedRegion&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;TargetIds:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;- !Ref TargetOUId&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Tags:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;- Key: Purpose&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Value: RegionCompliance&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;- Key: ManagedBy&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Value: CloudFormation&lt;/code&gt;&lt;br /&gt;&lt;br /&gt;&lt;code&gt;Outputs:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;PolicyId:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'ID of the created Service Control Policy'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Value: !Ref RegionDenySCP&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Export:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Name: !Sub '${AWS::StackName}-PolicyId'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp;PolicyArn:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Description: 'ARN of the created Service Control Policy'&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Value: !GetAtt RegionDenySCP.Arn&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp;Export:&lt;/code&gt;&lt;br /&gt;&lt;code&gt;&nbsp;&nbsp; &nbsp; &nbsp;Name: !Sub '${AWS::StackName}-PolicyArn'&lt;/code&gt;</code></pre> 
</div> 
<p>Other capabilities include compliance-related resources consistently, maintain audit trails of security configurations and ensure regulatory requirements are met across all accounts. For example, you can enable CouldTrail and deploy AWS Config rules across all the instance stacks managed by the StackSet.</p> 
<p>For both Security and Compliance incidents you can use StackSets to deploy automated response workflows, configure event notifications and implement remediation actions across your accounts and regions.</p> 
<h3>Import existing stacks into StackSets</h3> 
<p>A stack import operation can import existing stacks into new or existing StackSets, so that you can migrate existing stacks to a StackSet in one operation.</p> 
<h2>Solution Overview</h2> 
<p>This solution includes an AWS CodePipeline stack that creates a CI/CD pipeline to deploy our StackSet. This pipeline deploys an application stack containing the AWS CloudFormation StackSet with a monitoring dashboard in AWS CloudWatch.</p> 
<p><img alt="Solution overview" class="wp-image-24159 size-full aligncenter" height="496" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/03/StackSets-Page-2.drawio.png" width="973" /></p> 
<p style="text-align: center;"><strong>Figure 2. Solution overview</strong></p> 
<p>The following Amazon CloudWatch dashboard is an example of what you will in the target accounts after the StackSet is deployed:</p> 
<p><img alt="Dashboard example" class="aligncenter wp-image-24162 size-full" height="1298" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/03/Screenshot-2025-10-01-at-2.34.39 PM.png" width="1496" /></p> 
<p style="text-align: center;"><strong>Figure 3. Dashboard example</strong></p> 
<p>In the CI/CD pipeline, before running the deployment commands, it applies python security and quality code checks to ensure code quality and security and <a href="https://github.com/cdklabs/cdk-nag">cdk-nag</a> to ensure <a href="https://aws.amazon.com/architecture/well-architected">AWS Well Architected</a> best practices. You can find more details about these checks in the solution repository in README.md file.</p> 
<p>The solution includes 2 AWS CloudFormation stacks defined by in the AWS CDK application and a template for the StackSet that will be deployed in the target accounts and regions. This stack contains the monitoring dashboard that will be deployed en the target regions of each target account as a single unit.</p> 
<p>The idea of using <a href="https://aws.amazon.com/codepipeline/">AWS CodePipeline</a> with IaC is that development teams can define and share “pipelines-as-code” patterns for deploying their applications making it easy to add stages. This way, security and quality code testing can run any time you change the source code.</p> 
<p><img alt="Pipeline overview" class="aligncenter wp-image-24167 size-full" height="582" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/03/Screenshot-2025-11-03-at-10.05.13 AM.png" width="1143" /></p> 
<p style="text-align: center;"><strong>Figure 4. Pipeline overview</strong></p> 
<p>The best practice is to ensure shift-left: adding this checks to the earlier stages of the SDLC. You can accomplish this complementing your CI/CD pipeline with <a href="https://git-scm.com/docs/githooks">githooks</a> or IDE Plugins. For example with <a href="https://aws.amazon.com/q/developer/build/">Amazon Q Developer</a> IDE extension you can use the <a href="https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/code-reviews.html">review</a> function to analyze the security of your code locally.</p> 
<h2>Walkthrough</h2> 
<p>If you’d like to try this solution out yourself, visit the walkthrough in the corresponding GitHub repo: <a href="https://github.com/aws-cloudformation/aws-cloudformation-templates/tree/main/CloudFormation/StackSets-CDK">https://github.com/aws-cloudformation/aws-cloudformation-templates/tree/main/CloudFormation/StackSets-CDK</a></p> 
<p>To use the CI/CD pipeline just create a repository using any of the AWS CodeConnection git&nbsp;<a href="https://docs.aws.amazon.com/dtconsole/latest/userguide/supported-versions-connections.html">supported providers</a>&nbsp;and add the contents of the folder.&nbsp;All details are included in the README.md so you can always get the latest version of the code and how it works.</p> 
<h2>Conclusion</h2> 
<p>In this post, we showed how to use AWS CDK to deploy AWS CloudFormation StackSets to reduce operational overhead and ensure consistency, compliance and security across multiple regions and accounts. We also learned how to create a CI/CD pipeline to guarantee a robust DevSecOps cycle for our Infrastructure as Code.</p> 
<p>Now that we’ve explored the main concepts together, you can clone the example repository from the walkthrough section, follow the setup instructions, and customize the implementation to enhance AWS resources management across accounts and regions. Whether you’re managing a single account or multiple organizations, these practices can be adapted to your specific needs. Now that you learned the main concepts, go ahead and clone the example repository from walkthrough section, follow the setup instructions and customize the implementation to improve the AWS resources management across your accounts and regions.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/03/foto-AWS-slack.png" width="120" />
  </div> 
  <h3 class="lb-h4">Franco Abregu</h3> 
  <p style="text-align: left;">Franco Abregu is a Sr. Delivery Consultant – DevOps at AWS Professional Services based in Argentina. Franco focuses on transforming customers DevOps culture to improve developer productivity, operations, deployments and process standardization. His expertise includes CI/CD, Infrastructure as Code, software development and organizational adoption of DevOps culture.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/idriss-profile-cut-scaled.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Idriss Laouali Abdou</h3> 
  <p style="text-align: left;">Idriss Laouali Abdou is a Sr. Product Manager Technical for AWS Infrastructure-as-Code based in Seattle. He focuses on improving developer productivity through StackSets and CloudFormation Infrastructure provisioning experiences. Outside of work, you can find him creating educational content for thousands of students, cooking, or dancing..</p> 
 </div> 
</footer>
