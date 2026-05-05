---
title: "StackSets Deployment Strategies: Balancing Speed, Safety, and Scale to Optimize Deployments for Different Organizational Needs"
url: "https://aws.amazon.com/blogs/devops/stacksets-deployment-strategies-balancing-speed-safety-and-scale-to-optimize-deployments-for-different-organizational-needs/"
date: "Wed, 08 Oct 2025 18:03:57 +0000"
author: "Amar Meriche"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<p>AWS <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html">CloudFormation StackSets</a> enables organizations to deploy infrastructure consistently across multiple AWS accounts and regions. However, success depends on choosing the right deployment strategy that balances three critical factors: deployment speed, operational safety, and organizational scale. This guide explores proven StackSets deployment strategies specifically designed for multi-account infrastructure management.</p> 
<h2>Understanding StackSets Deployment Fundamentals</h2> 
<h3>What are StackSets Actually Used For?</h3> 
<p>Unlike single-account <a href="https://aws.amazon.com/fr/cloudformation/">AWS CloudFormation</a>&nbsp;templates, StackSets are specifically designed for&nbsp;<strong>multi-account infrastructure governance</strong>. Common use cases include <strong>Security baselines</strong> (deploying IAM policies, security groups, and access controls across all accounts), <strong>Compliance controls</strong> (rolling out <a href="https://aws.amazon.com/config/">AWS Config</a>&nbsp;rules, <a href="https://aws.amazon.com/cloudtrail/">AWS CloudTrail</a> configurations, and audit requirements),&nbsp;<strong>Organizational standards</strong> (establishing consistent VPC configurations, tagging policies, and naming conventions), <strong>Shared services</strong> (deploying monitoring solutions, logging infrastructure, and backup policies) or <strong>Cost management</strong> (implementing budget controls, cost allocation tags, and resource optimization policies)</p> 
<h3>The Multi-Account Challenge</h3> 
<p>Managing infrastructure across dozens or hundreds of AWS accounts presents unique challenges:</p> 
<p><code>Single Account (CFN Template) &nbsp; &nbsp; Multi-Account (StackSets)</code><br /> <code>&nbsp;&nbsp; &nbsp; &nbsp;App A &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Org Unit A (50 accounts)</code><br /> <code>&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |</code><br /> <code>&nbsp;&nbsp; [Deploy Once] &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [Deploy consistently across all]</code><br /> <code>&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |</code><br /> <code>&nbsp;&nbsp; &nbsp;Success/Fail &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Complex success/failure matrix</code></p> 
<p><span style="color: #808080;"><em>Multi account and multi region Cloudformation deployment complexity</em></span></p> 
<h3>The Speed-Safety-Scale Triangle</h3> 
<p>Every StackSets deployment strategy involves trade-offs: <strong>Speed</strong> (how quickly changes propagate across your organization), <strong>Safety</strong> (risk mitigation and failure containment) and <strong>Scale</strong> (ability to manage hundreds of accounts efficiently)</p> 
<h2>Prerequisites</h2> 
<p>Before implementing any of the deployment strategies described in this guide, ensure you have:</p> 
<ol> 
 <li>AWS CLI Installation 
  <ol> 
   <li>Install the latest version of AWS CLI by following the <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">AWS CLI installation guide</a></li> 
   <li>Verify installation with: aws –version</li> 
  </ol> </li> 
 <li>AWS Profile Configuration 
  <ol> 
   <li>Configure your AWS credentials using: aws configure</li> 
   <li>For details on configuration, see <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html">AWS CLI configuration basics</a></li> 
   <li>Ensure your profile has appropriate permissions for CloudFormation StackSets operations as described in <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs.html">AWS StackSets prerequisites</a></li> 
  </ol> </li> 
 <li>Proper Account Access The commands in this guide must be executed from either: 
  <ol> 
   <li>The management account of your AWS Organization</li> 
   <li>OR a delegated administrator account for CloudFormation</li> 
  </ol> </li> 
</ol> 
<p>For information on setting up a delegated administrator, see <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-delegated-admin.html">Register a delegated administrator</a></p> 
<p>Note: StackSets deployments using service-managed permissions cannot be performed from standalone accounts.</p> 
<p>Verify you’re using the correct account with:</p> 
<p><code>bash</code><br /> <code># For management account</code><br /> <code>aws organizations describe-organization</code><br /> <code># For delegated admin</code><br /> <code>aws cloudformation list-stack-sets —call-as DELEGATED_ADMIN</code></p> 
<p><em><span style="color: #808080;">AWS CLI to check the usage of an Organization and not a Standalone account</span></em></p> 
<h2>Core Deployment Strategies</h2> 
<p>As explained in the <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-bestpractices.html#w70aac15c41c11">StackSet documentation</a>:</p> 
<ul> 
 <li><em>“For a more conservative deployment, set <strong>Maximum Concurrent Accounts</strong> to 1, and <strong>Failure Tolerance</strong> to 0. Set your lowest-impact region to be first in the <strong>Region Order</strong> Start with one region.”</em></li> 
 <li><em>“For a faster deployment, increase the values of <strong>Maximum Concurrent Accounts</strong> and <strong>Failure Tolerance</strong> as needed. ”</em></li> 
</ul> 
<p>Based on the above, we are proposing below several deployment strategies, depending on the speed, safety and scale you want to achieve.</p> 
<h3>1. Sequential Deployment: Maximum Safety</h3> 
<p><strong>Use Case&nbsp;</strong>: Critical security updates, compliance requirements, first-time organizational rollouts</p> 
<p>Below are listed some possible use cases:</p> 
<ul> 
 <li><strong>Security baseline updates</strong>: New IAM policies affecting root access</li> 
 <li><strong>Compliance rollouts</strong>: SOX, HIPAA, or PCI-DSS control implementations</li> 
 <li><strong>Critical infrastructure changes</strong>: VPC security group modifications</li> 
 <li><strong>Organizational policy changes</strong>: New AWS Config rules for audit compliance</li> 
</ul> 
<p><strong>Implementation Example:</strong></p> 
<p>For this example, we will download the following template&nbsp;<a href="https://s3.amazonaws.com/cloudformation-stackset-sample-templates-us-east-1/ConfigRuleCloudtrailEnabled.yml">ConfigRuleCloudtrailEnabled.yml</a>&nbsp;from the <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-sampletemplates.html">Cloudformation sample library</a> in the AWS documentation to configure an AWS Config rule to determine if AWS CloudTrail is enabled and&nbsp;follow the next steps:</p> 
<p>Step 1: Create the StackSet</p> 
<p>With the AWS CLI:</p> 
<p><code># Create Stackset for security baseline</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code>aws cloudformation create-stack-set \</code><br /> <code>&nbsp;&nbsp;--stack-set-name security-baseline&nbsp;\</code><br /> <code>&nbsp;&nbsp;--template-body file://ConfigRuleCloudtrailEnabled.yml \</code><br /> <code>&nbsp;&nbsp;--capabilities CAPABILITY_NAMED_IAM \</code><br /> <code>&nbsp;&nbsp;--permission-model SERVICE_MANAGED \</code><br /> <code>&nbsp;&nbsp;--auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false \</code><br /> <code>&nbsp; --region us-east-1</code></p> 
<p><em><span style="color: #808080;">AWS CLI to create a security-baseline Stackset</span></em></p> 
<p>The expected response should be similar to the following :</p> 
<p><code>{"StacksetId": "security-baseline: ...."}</code></p> 
<p>Step 2: Create Stack Instances</p> 
<p>Before you launch the below command, you need to adjust the values of the following parameters:</p> 
<ul> 
 <li><strong>OrganizationalUnitIds</strong>: you must change the value “ou-test” in the below command line to the name of the target OU you want to deploy to. I recommend creating a new test OU&nbsp;<a href="https://docs.aws.amazon.com/organizations/latest/userguide/create_ou.html">in the console or via the CLI</a>&nbsp;for the purpose of this test.</li> 
 <li><strong>regions</strong>: if needed, change the “us-east-1 eu-west-1” value, here you need to list all the regions you want to deploy to. AWS Config must be active in the accounts/regions that you choose, otherwise you’ll get an error when deploying the Stack.</li> 
</ul> 
<p><code># Deploy security baseline to production accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1 and eu-west-1</code><br /> <code># SEQUENTIAL = One region at a time, sequentially </code><br /> <code># MaxConcurrentPercentage = Deploy to 5% of accounts at once</code><br /> <code># FailureTolerancePercentage = Stop on first failure</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>&nbsp;&nbsp;--stack-set-name security-baseline&nbsp;\</code><br /> <code>&nbsp;&nbsp;--deployment-targets OrganizationalUnitIds=ou-test\</code><br /> <code>&nbsp;&nbsp;--regions us-east-1 eu-west-1 \</code><br /> <code>&nbsp;&nbsp;--region us-east-1 \</code><br /> <code>&nbsp;&nbsp;--operation-preferences RegionConcurrencyType=SEQUENTIAL,MaxConcurrentPercentage=5,FailureTolerancePercentage=0</code></p> 
<p><em><span style="color: #808080;">AWS CLI to create security-baseline Stack Instances sequentially for maximum safety</span></em></p> 
<p>The CLI output should look like the following:</p> 
<p><code>{"OperationId":&nbsp;....}</code></p> 
<p><strong>Or</strong> create the StackSet and add the Stacks with the AWS Console:</p> 
<p>In the CloudFormation Console, click “Create StackSet”</p> 
<p><img alt="AWS CloudFormation Console: create a security-baseline Stackset" class="alignleft size-full wp-image-23966" height="319" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/1-Create-StackSet.png" width="1228" /></p> 
<p><span style="color: #808080;"><em>AWS CloudFormation Console: create a security-baseline Stackset</em></span></p> 
<p>Upload your template from S3 or from your computer and click Next:</p> 
<p><img alt="AWS CloudFormation Console: specify a template" class="size-full wp-image-23967 alignright" height="648" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/2-Upload-template-from-S3.png" width="1208" /></p> 
<p><em><span style="color: #808080;">AWS CloudFormation Console: specify a template</span></em></p> 
<p>Specify the StackSet name and parameters and click Next:</p> 
<p><img alt="AWS CloudFormation Console: specify the StackSet name and parameters" class="alignleft size-full wp-image-23968" height="730" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/3-StackSet-parameters.png" width="1171" /></p> 
<p><span style="color: #808080;"><em>AWS CloudFormation Console: specify the StackSet name and parameters</em></span></p> 
<p>Configure StackSet options and click Next:</p> 
<p><img alt="AWS CloudFormation Console: configure the StackSet options" class="size-full wp-image-23960 alignright" height="427" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/4-StackSet-options.png" width="1211" /></p> 
<p><em><span style="color: #808080;">AWS CloudFormation Console: configure the StackSet options</span></em></p> 
<p>Set deployment options and click Next:</p> 
<p><img alt="AWS CloudFormation Console: set deployment options" class="alignleft size-full wp-image-23961" height="670" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/5-Deployment-options.png" width="1200" /></p> 
<p><span style="color: #808080;"><em>AWS CloudFormation Console: set deployment options</em></span></p> 
<p><img alt="AWS CloudFormation Console: set deployment options" class="size-full wp-image-23962 alignright" height="596" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/6-Deployment-options.png" width="1193" /></p> 
<p><span style="color: #808080;"><em>AWS CloudFormation Console: set more deployment options</em></span></p> 
<p>Then Review and Submit.</p> 
<p>Not to overweight this blog, we’ll provide only this example of CLI output and Console screenshot, but the “Parallel Deployment” and “Balanced Approach” will be similar to this example. You just need to update the parameters for the different <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-concepts.html#stackset-ops-options">StackSet Operations options</a>.</p> 
<p>A real-world example would be a financial services company deploying new MFA requirements across 200 production accounts. They could use sequential deployment with 5 concurrency to ensure each batch was validated before proceeding.</p> 
<h3>2. Parallel Deployment: Maximum Speed</h3> 
<p>The Parallel Deployment is best for non-critical updates, development environments, routine maintenance</p> 
<p>Here are some possible use cases:</p> 
<ul> 
 <li><strong>Development account standardization</strong>: Rolling out new development tools</li> 
 <li><strong>Monitoring infrastructure</strong>: Deploying Amazon CloudWatch dashboards and alarms</li> 
 <li><strong>Cost optimization</strong>: Implementing automated resource cleanup policies</li> 
 <li><strong>Non-production updates</strong>: Updating development and staging environments</li> 
</ul> 
<p><strong>Implementation Example:</strong></p> 
<p>For this example, we will copy paste the .yml template&nbsp;from <a href="https://repost.aws/knowledge-center/cloudformation-iam-event-monitoring">this Re:Post article</a>&nbsp;about monitoring IAM events in a file called “monitoring-baseline.yml”, and use it in the following command lines.</p> 
<p>Step 1: Create the StackSet</p> 
<p><code># Create Stackset for monitoring baseline</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code>aws cloudformation create-stack-set \</code><br /> <code>--stack-set-name monitoring-baseline \</code><br /> <code>--template-body file://monitoring-baseline.yml \</code><br /> <code>--capabilities CAPABILITY_NAMED_IAM \</code><br /> <code>--permission-model SERVICE_MANAGED \</code><br /> <code>--auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false \</code><br /> <code>--region us-east-1</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to create a monitoring-baseline Stackset</em></span></p> 
<p>Step 2: Create Stack Instances</p> 
<p>Just like in the previous example, before you launch the below command, you need to adjust the values of the&nbsp;OrganizationalUnitIds and regions parameters.</p> 
<p><code># Deploy monitoring baseline to dev and sandbox accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1 and eu-west-1</code><br /> <code># PARALLEL = Deployment in parallel</code><br /> <code># MaxConcurrentPercentage = Deploy to 80% of accounts at once</code><br /> <code># FailureTolerancePercentage = Tolerate failures in 20% of accounts</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>--stack-set-name monitoring-baseline \</code><br /> <code>--deployment-targets OrganizationalUnitIds=ou-development,ou-sandbox \</code><br /> <code>--regions us-east-1 eu-west-1 \</code><br /> <code>--region us-east-1 \</code><br /> <code>--operation-preferences RegionConcurrencyType=PARALLEL,MaxConcurrentPercentage=80,FailureTolerancePercentage=20</code></p> 
<p><em><span style="color: #808080;">AWS CLI to create monitoring-baseline Stack Instances in parallel with high value for max concurrent percentage for maximum speed</span></em></p> 
<h3>3. Progressive Deployment: Balanced Approach or Multi Phase Approach (Recommended)</h3> 
<p>For most production scenarios with moderate risk tolerance, it is recommended to use a Balanced Approach, or Multi-Phase Implementation.</p> 
<p><strong>Balanced Approach</strong></p> 
<p>For this example, to make it easier, you can create a copy of “monitoring-baseline.yml” created previously, and name it “balanced-template.yml”.</p> 
<p><code>cp&nbsp;monitoring-baseline.yml&nbsp;balanced-template.yml</code></p> 
<p><span style="color: #808080;"><em>bash command to copy the monitoring-baseline.yml file to balanced-template.yml</em></span></p> 
<p>Then you can use it in the following command lines.</p> 
<p>Step 1: Create the StackSet</p> 
<p><code># Create Stackset for a balanced creation</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code>aws cloudformation create-stack-set \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--template-body file://balanced-template.yml \</code><br /> <code>--capabilities CAPABILITY_NAMED_IAM \</code><br /> <code>--permission-model SERVICE_MANAGED \</code><br /> <code>--auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false \</code><br /> <code>--region us-east-1</code></p> 
<p><em><span style="color: #808080;">AWS CLI to create a balanced-deployment Stackset</span></em></p> 
<p>Step 2: Create Stack Instances</p> 
<p>You need to adjust the values of the&nbsp;OrganizationalUnitIds and regions parameters.</p> 
<p><code># Deploy monitoring baseline to production accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1, eu-west-1 and ap-southeast-1</code><br /> <code># PARALLEL = Deployment in parallel</code><br /> <code># MaxConcurrentPercentage = Deploy to 25% of accounts at once</code><br /> <code># FailureTolerancePercentage = Tolerate failures in 8% of accounts</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--deployment-targets OrganizationalUnitIds=ou-development,ou-sandbox \</code><br /> <code>--regions us-east-1 eu-west-1 ap-southeast-1 \</code><br /> <code>--region us-east-1 \</code><br /> <code>--operation-preferences RegionConcurrencyType=PARALLEL,MaxConcurrentPercentage=25,FailureTolerancePercentage=8</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to create balanced-deployment Stack Instances in parallel with low max concurrent percentage for a balanced deployment</em></span></p> 
<p><strong>Multi-Phase Implementation:</strong></p> 
<p>Step 1: Create the StackSet</p> 
<p><code># Create Stackset for a balanced creation</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code>aws cloudformation create-stack-set \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--template-body file://balanced-template.yml \</code><br /> <code>--capabilities CAPABILITY_NAMED_IAM \</code><br /> <code>--permission-model SERVICE_MANAGED \</code><br /> <code>--auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false \</code><br /> <code>--region us-east-1</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to create a balanced-deployment Stackset</em></span></p> 
<p><strong>Phase 1: Pilot Accounts (10% of target)</strong></p> 
<p>Phase 1: Create Pilot Stack Instances</p> 
<p>You need to adjust the values of the&nbsp;OrganizationalUnitIds and regions parameters.</p> 
<p><code># Deploy monitoring baseline to production accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1</code><br /> <code># SEQUENTIAL = Deployment in sequence</code><br /> <code># MaxConcurrentPercentage = 100% Deploy full speed for small pilot</code><br /> <code># FailureTolerancePercentage = Zero tolerance in pilot</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--deployment-targets Accounts=pilot-account-1,pilot-account-2 \</code><br /> <code>--regions us-east-1 \</code><br /> <code>--region us-east-1 \</code><br /> <code>--operation-preferences RegionConcurrencyType=SEQUENTIAL,MaxConcurrentPercentage=100,FailureTolerancePercentage=0</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to create balanced-deployment Stack Instances sequentially for maximum safety in Pilot accounts</em></span></p> 
<p>Wait for Pilot validation before proceeding to Phase 2</p> 
<p><strong>Phase 2: Early Adopter OUs (30% of target)</strong></p> 
<p>Phase 2: Create Early Adopter Stack Instances</p> 
<p>You need to adjust the values of the&nbsp;OrganizationalUnitIds and regions parameters.</p> 
<p><code># Deploy monitoring baseline to production accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1, eu-west-1</code><br /> <code># PARALLEL = Deployment in parallel</code><br /> <code># MaxConcurrentPercentage = Deploy to 25% of accounts at once</code><br /> <code># FailureTolerancePercentage = Tolerate failures in 5% of accounts</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--deployment-targets OrganizationalUnitIds=ou-early-adopter \</code><br /> <code>--regions us-east-1 \</code><br /> <code>--region us-east-1 eu-west-1 \</code><br /> <code>--operation-preferences RegionConcurrencyType=PARALLEL,MaxConcurrentPercentage=25,FailureTolerancePercentage=5</code></p> 
<p><em><span style="color: #808080;">AWS CLI to create balanced-deployment Stack Instances in parallel with low max concurrent percentage for a balanced deployment in Early Adopter OU</span></em></p> 
<p>Wait for Early Adopter validation before proceeding to Phase 3</p> 
<p><strong>Phase 3: Full Deployment (Remaining 60%)</strong></p> 
<p>Phase 3: Full Deployment</p> 
<p>You need to adjust the values of the&nbsp;OrganizationalUnitIds and regions parameters.</p> 
<p><code># Deploy monitoring baseline to production accounts</code><br /> <code># StackSet operation managed from us-east-1</code><br /> <code># Deployed to regions us-east-1, eu-west-1 and ap-southeast-1</code><br /> <code># PARALLEL = Deployment in parallel</code><br /> <code># MaxConcurrentPercentage = Deploy to 40% of accounts at once for higher speed after validation</code><br /> <code># FailureTolerancePercentage = Tolerate failures in 10% of accounts for moderate tolerance</code><br /> <code>aws cloudformation create-stack-instances \</code><br /> <code>--stack-set-name balanced-deployment \</code><br /> <code>--deployment-targets OrganizationalUnitIds=ou-standard-prod,ou-legacy-prod \</code><br /> <code>--regions us-east-1 \</code><br /> <code>--region us-east-1 eu-west-1 ap-southeast-1 \</code><br /> <code>--operation-preferences RegionConcurrencyType=PARALLEL,MaxConcurrentPercentage=25,FailureTolerancePercentage=5</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to create balanced-deployment Stack Instances in parallel with low max concurrent percentage for a balanced deployment in the remaining OUs</em></span></p> 
<h3>Using Step Functions for Orchestration</h3> 
<p>AWS Step Functions provides a serverless workflow service that can <a href="https://aws.amazon.com/blogs/mt/aws-cloudformation-stackset-orchestration-automated-deployment-using-aws-step-functions/">orchestrate StackSets deployments</a> with advanced control flow, error handling, and state management capabilities. This approach enhances your multi-account deployments with features not available through standard StackSets operations alone.</p> 
<p><strong>Some of the Key Benefits include:</strong></p> 
<ul> 
 <li><strong>Advanced Deployment Orchestration</strong>: Coordinate multi-phase rollouts with validation gates</li> 
 <li><strong>Human Approval Workflows</strong>: Implement manual approval steps for critical changes</li> 
 <li><strong>Enhanced Error Handling</strong>: Define sophisticated retry policies and fallback mechanisms</li> 
 <li><strong>Visual Monitoring</strong>: Track deployment progress through the Step Functions visual console</li> 
</ul> 
<p><strong>Real-World Use Case: Compliance Control Rollout</strong></p> 
<p>In regulated industries, AWS Step Functions enables a phased approach that combines automation with necessary governance. For instance, you can:</p> 
<ol> 
 <li>Deploy compliance controls to test accounts</li> 
 <li>Run automated validation and generate compliance reports</li> 
 <li>Obtain manual approval from compliance team</li> 
 <li>Deploy to production accounts with comprehensive monitoring</li> 
</ol> 
<p>This approach ensures consistent governance while maintaining the complete audit trail required for regulatory compliance.</p> 
<h2>Monitoring and Optimization</h2> 
<p>AWS CloudFormation StackSets do not have extensive built-in Amazon CloudWatch metrics specifically designed for monitoring StackSet operations and health. This is actually why the monitoring implementation in our blog post is valuable.</p> 
<p>Here’s what AWS does and doesn’t provide out of the box:</p> 
<p><strong>What AWS provides natively:</strong></p> 
<ul> 
 <li>Basic AWS API call metrics via AWS CloudTrail (which show that operations happened but don’t track success rates or performance)</li> 
 <li>General service quotas and throttling metrics for CloudFormation as a whole</li> 
 <li>CloudFormation provides some metrics for individual stacks, but not consolidated StackSet-specific metrics</li> 
</ul> 
<p><strong>What requires custom implementation (as in our blog post):</strong></p> 
<ul> 
 <li>Success rate metrics for StackSet operations across accounts</li> 
 <li>Deployment completion time tracking</li> 
 <li>Configuration drift detection and monitoring</li> 
 <li>Account-specific failure analysis</li> 
 <li>Comprehensive dashboards that show StackSet health across your organization</li> 
</ul> 
<p>The code in our blog post demonstrates how to implement the success rate custom metrics by:</p> 
<ol> 
 <li>Gathering data from the CloudFormation API about StackSet operations</li> 
 <li>Calculating the success rate metrics for StackSet deployments</li> 
 <li>Creating custom Amazon CloudWatch metrics in a custom namespace (like “StackSetMonitoring”)</li> 
 <li>Setting up alerts for issues</li> 
</ol> 
<p>This explains why organizations need to implement custom monitoring solutions like the one shown in our blog post rather than relying solely on built-in metrics.</p> 
<h3>Automated Monitoring Implementation: example of a custom metric to monitor the StackSet operations success rate</h3> 
<p>The following AWS Cloudformation template provides real-time monitoring and alerting for AWS CloudFormation StackSet operations through automated infrastructure deployment. This solution creates a complete monitoring system using a AWS Lambda function, Amazon EventBridge rules, Amazon SNS notifications, and Amazon CloudWatch dashboards to track StackSet success and failure rates. The core Lambda function named StackSetMonitor continuously monitors all active StackSets in your account, calculating success rates and publishing custom metrics to Amazon CloudWatch under the StackSetMonitoring namespace.</p> 
<p>Below you’ll find a few example of possible custom metrics that could be implemented based on this AWS Cloudformation template:</p> 
<ul> 
 <li>Count of all operations (CREATE, UPDATE, DELETE) per StackSet over time periods</li> 
 <li>Number of stack instances with configuration drift (requires additional API calls)</li> 
 <li>Average time taken for StackSet operations to complete</li> 
 <li>Rate of StackSet operations to identify peak usage times</li> 
 <li>Number of individual stack instances that failed during operations</li> 
 <li>Number of retried operations (indicates infrastructure issues)</li> 
 <li>…</li> 
</ul> 
<p>Here’s the StackSetMonitor.yml CloudFormation Template:</p> 
<pre><code class="lang-yaml"># StackSetMonitor.yml 
# CFN template for monitoring AWS CloudFormation StackSet operations with real-time alerts, metrics, and dashboards.

AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation template for StackSet operation monitoring using CloudWatch and SNS'

Parameters:
  StackSetName:
    Type: String
    Description: 'Name of the StackSet to monitor'
    Default: 'security-baseline'
    MinLength: 1
    MaxLength: 128
    AllowedPattern: '[a-zA-Z][-a-zA-Z0-9]*'
    ConstraintDescription: 'Must be a valid StackSet name (1-128 characters, alphanumeric and hyphens, must start with a letter)'
  
  VpcId:
    Type: String
    Description: 'VPC ID where the Lambda function will be deployed (leave empty to create new VPC)'
    Default: ''
  
  SubnetIds:
    Type: CommaDelimitedList
    Description: 'List of subnet IDs for the Lambda function (leave empty to create new subnets)'
    Default: ''
    
  SecurityGroupIds:
    Type: CommaDelimitedList
    Description: 'List of security group IDs for the Lambda function (leave empty to create new security group)'
    Default: ''

Conditions:
  CreateVPC: !Equals [!Ref VpcId, '']
  CreateVPCAndSubnets: !And [!Equals [!Ref VpcId, ''], !Equals [!Join [',', !Ref SubnetIds], '']]
  HasCustomSecurityGroups: !Not [!Equals [!Join [',', !Ref SecurityGroupIds], '']]
  
Resources:
  # KMS Key for CloudWatch Logs encryption
  LogsKMSKey:
    Type: AWS::KMS::Key
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties:
      Description: 'KMS Key for StackSet Monitor CloudWatch Logs and Lambda environment variable encryption'
      EnableKeyRotation: true
      KeyPolicy:
        Version: '2012-10-17'
        Statement:
          - Sid: Enable IAM User Permissions
            Effect: Allow
            Principal:
              AWS: !Sub 'arn:${AWS::Partition}:iam::${AWS::AccountId}:root'
            Action: 'kms:*'
            Resource: '*'
          - Sid: Allow CloudWatch Logs
            Effect: Allow
            Principal:
              Service: !Sub 'logs.${AWS::Region}.amazonaws.com'
            Action:
              - 'kms:Encrypt'
              - 'kms:Decrypt'
              - 'kms:ReEncrypt*'
              - 'kms:GenerateDataKey*'
              - 'kms:DescribeKey'
            Resource: '*'
            Condition:
              ArnEquals:
                'kms:EncryptionContext:aws:logs:arn': 
                  - !Sub 'arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/StackSetMonitor'
                  - !Sub 'arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/cloudformation/stacksets'
          - Sid: Allow Lambda Service
            Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action:
              - 'kms:Encrypt'
              - 'kms:Decrypt'
              - 'kms:ReEncrypt*'
              - 'kms:GenerateDataKey*'
              - 'kms:DescribeKey'
            Resource: '*'

  LogsKMSKeyAlias:
    Type: AWS::KMS::Alias
    Properties:
      AliasName: alias/stackset-monitor-logs
      TargetKeyId: !Ref LogsKMSKey

  # VPC Resources (created when no existing VPC is provided)
  StackSetMonitorVPC:
    Type: AWS::EC2::VPC
    Condition: CreateVPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: StackSetMonitor-VPC
        - Key: Purpose
          Value: VPC for StackSet Monitor Lambda function


  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: StackSetMonitor-Private-Subnet-1
        - Key: Purpose
          Value: Private subnet for StackSet Monitor Lambda

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: StackSetMonitor-Private-Subnet-2
        - Key: Purpose
          Value: Private subnet for StackSet Monitor Lambda

  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      Tags:
        - Key: Name
          Value: StackSetMonitor-Private-RT-1

  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      Tags:
        - Key: Name
          Value: StackSetMonitor-Private-RT-2

  PrivateSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: CreateVPC
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      SubnetId: !Ref PrivateSubnet1

  PrivateSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: CreateVPC
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      SubnetId: !Ref PrivateSubnet2

  # VPC Endpoints for AWS Services (no internet access needed)
  CloudFormationVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.cloudformation
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - cloudformation:ListStackSets
              - cloudformation:ListStackSetOperations
              - cloudformation:ListStackInstances
              - cloudformation:DescribeStackInstance
              - cloudformation:DescribeStacks
              - cloudformation:GetTemplate
            Resource: '*'

  CloudWatchVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.monitoring
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - cloudwatch:PutMetricData
            Resource: '*'

  SNSVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.sns
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sns:Publish
            Resource: '*'

  EventsVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.events
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - events:PutEvents
            Resource: '*'

  LogsVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.logs
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - logs:CreateLogGroup
              - logs:CreateLogStream
              - logs:PutLogEvents
            Resource: '*'

  SQSVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.sqs
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sqs:SendMessage
            Resource: '*'

  STSVPCEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Condition: CreateVPC
    Properties:
      VpcId: !Ref StackSetMonitorVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.sts
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref VPCEndpointSecurityGroup
      PrivateDnsEnabled: true
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sts:AssumeRole
              - sts:GetCallerIdentity
              - sts:AssumeRoleWithWebIdentity
            Resource: '*'

  # Security Group for Lambda function
  LambdaSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for StackSet Monitor Lambda function
      VpcId: !If
        - CreateVPC
        - !Ref StackSetMonitorVPC
        - !Ref VpcId
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 10.0.0.0/16
          Description: HTTPS to VPC Endpoints
        - IpProtocol: tcp
          FromPort: 53
          ToPort: 53
          CidrIp: 10.0.0.0/16
          Description: DNS TCP to VPC for name resolution
        - IpProtocol: udp
          FromPort: 53
          ToPort: 53
          CidrIp: 10.0.0.0/16
          Description: DNS UDP to VPC for name resolution
      Tags:
        - Key: Name
          Value: StackSetMonitor-Lambda-SG
        - Key: Purpose
          Value: Security group for StackSet Monitor Lambda

  VPCEndpointSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Condition: CreateVPC
    Properties:
      GroupDescription: Security group for VPC Endpoints
      VpcId: !Ref StackSetMonitorVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          SourceSecurityGroupId: !Ref LambdaSecurityGroup
          Description: HTTPS from Lambda security group
        - IpProtocol: tcp
          FromPort: 53
          ToPort: 53
          SourceSecurityGroupId: !Ref LambdaSecurityGroup
          Description: DNS TCP from Lambda security group
        - IpProtocol: udp
          FromPort: 53
          ToPort: 53
          SourceSecurityGroupId: !Ref LambdaSecurityGroup
          Description: DNS UDP from Lambda security group
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 10.0.0.0/16
          Description: HTTPS outbound within VPC
        - IpProtocol: tcp
          FromPort: 53
          ToPort: 53
          CidrIp: 10.0.0.0/16
          Description: DNS TCP outbound within VPC
        - IpProtocol: udp
          FromPort: 53
          ToPort: 53
          CidrIp: 10.0.0.0/16
          Description: DNS UDP outbound within VPC
      Tags:
        - Key: Name
          Value: StackSetMonitor-VPCEndpoint-SG
        - Key: Purpose
          Value: Security group for VPC Endpoints

  # Dead Letter Queue for Lambda function
  StackSetMonitorDLQ:
    Type: AWS::SQS::Queue
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties:
      QueueName: StackSetMonitor-DLQ
      MessageRetentionPeriod: 1209600  # 14 days
      KmsMasterKeyId: alias/aws/sqs
      Tags:
        - Key: Purpose
          Value: Dead Letter Queue for StackSet Monitor Lambda

  StackSetAlertsTopic:
    Type: AWS::SNS::Topic
    Properties: 
      TopicName: StackSetAlerts
      DisplayName: StackSet Monitoring Alerts
      KmsMasterKeyId: alias/aws/sns
  
  StackSetLogGroup:
    Type: AWS::Logs::LogGroup
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties: 
      LogGroupName: /aws/cloudformation/stacksets
      RetentionInDays: 30
      KmsKeyId: !GetAtt LogsKMSKey.Arn

  LambdaLogGroup:
    Type: AWS::Logs::LogGroup
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties:
      LogGroupName: /aws/lambda/StackSetMonitor
      RetentionInDays: 30
      KmsKeyId: !GetAtt LogsKMSKey.Arn
  
  StackSetMonitoringDashboard:
    Type: AWS::CloudWatch::Dashboard
    Properties:
      DashboardName: StackSetMonitoring
      DashboardBody: !Sub |
        {
          "widgets": [
            {
              "type": "metric",
              "width": 24,
              "height": 8,
              "properties": {
                "metrics": [
                  [ "StackSetMonitoring", "SuccessRate", "StackSetName", "${StackSetName}" ]
                ],
                "region": "${AWS::Region}",
                "title": "StackSet Operations",
                "period": 300,
                "stat": "Average"
              }
            },
            {
              "type": "log",
              "width": 24,
              "height": 6,
              "properties": {
                "query": "SOURCE '/aws/lambda/StackSetMonitor' | fields @timestamp, @message\n| sort @timestamp desc\n| limit 20",
                "region": "${AWS::Region}",
                "title": "Latest StackSet Monitor Logs",
                "view": "table"
              }
            }
          ]
        }
  
  # Consolidated rule to catch ALL StackSet events for comprehensive monitoring
  AllStackSetOperationsRule:
    Type: AWS::Events::Rule
    Properties:
      Name: AllStackSetOperationsRule
      Description: "Rule for monitoring all CloudFormation StackSet operations with failure notifications"
      EventPattern: {source: ["aws.cloudformation"], detail-type: ["CloudFormation StackSet Operation Status Change"]}
      State: ENABLED
      Targets:
        - Id: ProcessAllEvents
          Arn: !GetAtt StackSetMonitorLambda.Arn
        - Id: NotifyFailure
          Arn: !Ref StackSetAlertsTopic
          InputTransformer:
            InputPathsMap:
              "stackSetId": "$.detail.stack-set-id"
              "operationId": "$.detail.operation-id"
              "status": "$.detail.status"
              "time": "$.time"
            InputTemplate: '"StackSet Event: ID: &lt;stackSetId&gt;, Op: &lt;operationId&gt;, Status: &lt;status&gt;, Time: &lt;time&gt;"'

  StackSetMonitorLambda:
    Type: AWS::Lambda::Function
    DependsOn: LambdaLogGroup
    Properties:
      FunctionName: StackSetMonitor
      Handler: index.lambda_handler
      Role: !GetAtt StackSetMonitorRole.Arn
      Runtime: python3.12
      Timeout: 300
      MemorySize: 512
      ReservedConcurrentExecutions: 1
      DeadLetterConfig:
        TargetArn: !GetAtt StackSetMonitorDLQ.Arn
      VpcConfig:
        SecurityGroupIds: !If
          - HasCustomSecurityGroups
          - !Ref SecurityGroupIds
          - - !Ref LambdaSecurityGroup
        SubnetIds: !If
          - CreateVPCAndSubnets
          - - !Ref PrivateSubnet1
            - !Ref PrivateSubnet2
          - !Ref SubnetIds
      KmsKeyArn: !GetAtt LogsKMSKey.Arn
      Code:
        ZipFile: |
          import boto3
          import json
          import os
          import logging
          import time
          import datetime
          from typing import Dict, Any, Optional
          
          # Custom JSON encoder to handle datetime objects
          class DateTimeEncoder(json.JSONEncoder):
              def default(self, obj):
                  if isinstance(obj, datetime.datetime):
                      return obj.isoformat()
                  return super().default(obj)
          
          # Set up logging with more details
          logger = logging.getLogger()
          logger.setLevel(logging.INFO)
          
          # Log initialization to verify Lambda is loading correctly
          print("StackSetMonitor Lambda initializing...")
          
          def validate_event(event: Dict[str, Any]) -&gt; bool:
              """Validate the incoming event structure"""
              if not isinstance(event, dict):
                  logger.error("Event must be a dictionary")
                  return False
              
              # If it's an EventBridge event, validate required fields
              if 'detail' in event:
                  detail = event.get('detail', {})
                  if not isinstance(detail, dict):
                      logger.error("Event detail must be a dictionary")
                      return False
                  
                  # Validate StackSet event structure
                  if 'stack-set-id' in detail:
                      stack_set_id = detail.get('stack-set-id')
                      if not isinstance(stack_set_id, str) or not stack_set_id.strip():
                          logger.error("stack-set-id must be a non-empty string")
                          return False
                      
                      # Validate operation-id if present
                      operation_id = detail.get('operation-id')
                      if operation_id is not None and not isinstance(operation_id, str):
                          logger.error("operation-id must be a string if provided")
                          return False
                      
                      # Validate status if present
                      status = detail.get('status')
                      if status is not None and not isinstance(status, str):
                          logger.error("status must be a string if provided")
                          return False
              
              return True
          
          def validate_context(context: Any) -&gt; bool:
              """Validate the Lambda context object"""
              if context is None:
                  logger.error("Context cannot be None")
                  return False
              
              # Check for required context attributes
              required_attrs = ['function_name', 'function_version', 'invoked_function_arn', 'memory_limit_in_mb']
              for attr in required_attrs:
                  if not hasattr(context, attr):
                      logger.error(f"Context missing required attribute: {attr}")
                      return False
              
              return True
          
          def sanitize_string(value: str, max_length: int = 255) -&gt; str:
              """Sanitize and truncate string inputs"""
              if not isinstance(value, str):
                  return str(value)[:max_length]
              return value.strip()[:max_length]
          
          def lambda_handler(event: Dict[str, Any], context: Any) -&gt; Dict[str, Any]:
              """Main Lambda handler function for StackSet monitoring with input validation"""
              
              # Input validation
              if not validate_event(event):
                  return {
                      "statusCode": 400,
                      "body": json.dumps({
                          "status": "error",
                          "message": "Invalid event structure"
                      }, cls=DateTimeEncoder)
                  }
              
              if not validate_context(context):
                  return {
                      "statusCode": 400,
                      "body": json.dumps({
                          "status": "error",
                          "message": "Invalid context object"
                      }, cls=DateTimeEncoder)
                  }
              
              # Log the validated event for debugging
              logger.info(f"Event received: {json.dumps(event, cls=DateTimeEncoder)}")
              logger.info(f"Function: {context.function_name}, Version: {context.function_version}")
              
              try:
                  cf = boto3.client('cloudformation')
                  cw = boto3.client('cloudwatch')
                  
                  # Log that we're starting processing
                  logger.info(f"Starting StackSet monitoring at {time.time()}")
                  
                  # Check if this is an event from EventBridge
                  if 'detail' in event and 'stack-set-id' in event.get('detail', {}):
                      detail = event['detail']
                      stack_set_id = sanitize_string(detail['stack-set-id'])
                      operation_id = sanitize_string(detail.get('operation-id', 'N/A'))
                      status = sanitize_string(detail.get('status', 'N/A'))
                      
                      # Validate stack_set_id format
                      if not stack_set_id or len(stack_set_id) &gt; 128:
                          logger.error(f"Invalid stack_set_id: {stack_set_id}")
                          return {
                              "statusCode": 400,
                              "body": json.dumps({
                                  "status": "error",
                                  "message": "Invalid stack_set_id format"
                              }, cls=DateTimeEncoder)
                          }
                      
                      # Log the StackSet operation with additional context
                      logger.info(f"Processing StackSet event - ID: {stack_set_id}, Op: {operation_id}, Status: {status}")
                      
                      # Extract stack set name from the ID
                      stack_set_name = stack_set_id.split('/')[-1] if '/' in stack_set_id else stack_set_id
                      stack_set_name = sanitize_string(stack_set_name, 128)
                      logger.info(f"Extracted StackSet name: {stack_set_name}")
                  
                  # Always gather metrics regardless of event type
                  # Get all active StackSets
                  stack_sets_response = cf.list_stack_sets(Status='ACTIVE')
                  stack_sets = stack_sets_response.get('Summaries', [])
                  
                  if not isinstance(stack_sets, list):
                      logger.error("Invalid response from list_stack_sets")
                      return {
                          "statusCode": 500,
                          "body": json.dumps({
                              "status": "error",
                              "message": "Invalid CloudFormation API response"
                          }, cls=DateTimeEncoder)
                      }
                  
                  logger.info(f"Found {len(stack_sets)} active StackSets")
                  
                  for stack_set in stack_sets:
                      if not isinstance(stack_set, dict) or 'StackSetName' not in stack_set:
                          logger.warning(f"Skipping invalid stack_set entry: {stack_set}")
                          continue
                      
                      stack_set_name = sanitize_string(stack_set['StackSetName'], 128)
                      logger.info(f"Processing StackSet: {stack_set_name}")
                      
                      try:
                          operations = cf.list_stack_set_operations(StackSetName=stack_set_name, MaxResults=5)
                          
                          # Validate operations response
                          if not isinstance(operations, dict):
                              logger.error(f"Invalid operations response for {stack_set_name}")
                              continue
                          
                          # Calculate success rate
                          successes = 0
                          operations_list = operations.get('Summaries', [])
                          
                          if not isinstance(operations_list, list):
                              logger.error(f"Invalid operations list for {stack_set_name}")
                              continue
                          
                          total_ops = len(operations_list)
                          logger.info(f"Found {total_ops} recent operations for {stack_set_name}")
                          
                          for op in operations_list:
                              if isinstance(op, dict) and op.get('Status') == 'SUCCEEDED':
                                  successes += 1
                          
                          success_rate = (successes / total_ops * 100) if total_ops &gt; 0 else 100
                          
                          # Validate success_rate is within expected bounds
                          if not (0 &lt;= success_rate &lt;= 100):
                              logger.error(f"Invalid success_rate calculated: {success_rate}")
                              continue
                          
                          # Publish metrics to CloudWatch
                          cw.put_metric_data(
                              Namespace='StackSetMonitoring',
                              MetricData=[
                                  {'MetricName': 'SuccessRate', 'Value': success_rate, 
                                   'Dimensions': [{'Name': 'StackSetName', 'Value': stack_set_name}]}
                              ]
                          )
                          
                          logger.info(f"Published metrics for {stack_set_name}: Success Rate = {success_rate}%")
                      except Exception as e:
                          logger.error(f"Error processing StackSet {stack_set_name}: {str(e)}")
                  
                  return {
                      "statusCode": 200,
                      "body": json.dumps({
                          "status": "completed",
                          "message": f"Processed {len(stack_sets)} StackSets"
                      }, cls=DateTimeEncoder)
                  }
                  
              except Exception as e:
                  logger.error(f"Error in Lambda function: {str(e)}")
                  # Return a proper response even on error
                  return {
                      "statusCode": 500,
                      "body": json.dumps({
                          "status": "error",
                          "message": str(e)
                      }, cls=DateTimeEncoder)
                  }
  
  # Managed IAM Policies
  CloudFormationAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: 'Policy for CloudFormation and CloudWatch access for StackSet Monitor'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - cloudformation:ListStackSets
              - cloudformation:ListStackSetOperations
              - cloudformation:ListStackInstances
              - cloudformation:DescribeStackInstance
            Resource: 
              - !Sub "arn:${AWS::Partition}:cloudformation:${AWS::Region}:${AWS::AccountId}:stackset/*"
              - !Sub "arn:${AWS::Partition}:cloudformation:${AWS::Region}:${AWS::AccountId}:stackset-target/*"
          - Effect: Allow
            Action:
              - cloudwatch:PutMetricData
            Resource: "*"
            Condition:
              StringEquals:
                "cloudwatch:namespace": "StackSetMonitoring"
          - Effect: Allow
            Action:
              - sns:Publish
            Resource: !Ref StackSetAlertsTopic

  EventsAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: 'Policy for EventBridge access for StackSet Monitor'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - events:PutEvents
            Resource: !Sub "arn:${AWS::Partition}:events:${AWS::Region}:${AWS::AccountId}:event-bus/default"

  LogsAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: 'Policy for CloudWatch Logs access for StackSet Monitor'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - logs:CreateLogGroup
              - logs:CreateLogStream
              - logs:PutLogEvents
            Resource: 
              - !Sub "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/StackSetMonitor"
              - !Sub "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/StackSetMonitor:*"
              - !Sub "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/cloudformation/stacksets"
              - !Sub "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/cloudformation/stacksets:*"

  DLQAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: 'Policy for Dead Letter Queue access for StackSet Monitor'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - sqs:SendMessage
            Resource: !GetAtt StackSetMonitorDLQ.Arn

  StackSetMonitorRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
        - !Ref CloudFormationAccessPolicy
        - !Ref EventsAccessPolicy
        - !Ref LogsAccessPolicy
        - !Ref DLQAccessPolicy

  # Permissions for event rules to invoke Lambda
  AllOperationsRuleLambdaPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref StackSetMonitorLambda
      Action: lambda:InvokeFunction
      Principal: events.amazonaws.com
      SourceArn: !GetAtt AllStackSetOperationsRule.Arn
  
  # Using a one minute schedule for testing, but you can change this value
  StackSetMonitorSchedule:
    Type: AWS::Events::Rule
    Properties:
      Name: RegularStackSetMonitoring
      Description: "Triggers Lambda function every 1 minute to check StackSet operations"
      ScheduleExpression: "rate(1 minute)"
      State: ENABLED
      Targets:
        - Id: RunMonitor
          Arn: !GetAtt StackSetMonitorLambda.Arn
  
  ScheduleLambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref StackSetMonitorLambda
      Action: lambda:InvokeFunction
      Principal: events.amazonaws.com
      SourceArn: !GetAtt StackSetMonitorSchedule.Arn
  
  StackSetSuccessRateAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: "Alarm when StackSet operation success rate is low"
      MetricName: SuccessRate
      Namespace: "StackSetMonitoring"
      Statistic: Average
      Period: 300
      EvaluationPeriods: 3
      DatapointsToAlarm: 2
      Threshold: 80
      ComparisonOperator: LessThanThreshold
      AlarmActions: [!Ref StackSetAlertsTopic]
      Dimensions: [{Name: StackSetName, Value: !Ref StackSetName}]

Outputs:
  SNSTopicArn: 
    Description: The ARN of the SNS topic for alerts
    Value: !Ref StackSetAlertsTopic
  DashboardURL: 
    Description: URL to the CloudWatch Dashboard
    Value: !Sub https://console.aws.amazon.com/cloudwatch/home?region=${AWS::Region}#dashboards:name=StackSetMonitoring
  LambdaLogGroupName:
    Description: Name of the CloudWatch Log Group for Lambda logs
    Value: !Ref LambdaLogGroup
  DeadLetterQueueArn:
    Description: ARN of the Dead Letter Queue for Lambda function failures
    Value: !GetAtt StackSetMonitorDLQ.Arn
  DeadLetterQueueURL:
    Description: URL of the Dead Letter Queue for monitoring failed Lambda executions
    Value: !Ref StackSetMonitorDLQ
  TestLambdaCommand:
    Description: Command to manually test the Lambda function
    Value: !Sub "aws lambda invoke --function-name ${StackSetMonitorLambda} --payload '{}' response.json &amp;&amp; cat response.json"
  LambdaFunctionArn:
    Description: ARN of the Lambda function configured with VPC
    Value: !GetAtt StackSetMonitorLambda.Arn
  LambdaSecurityGroupId:
    Description: Security Group ID created for the Lambda function
    Value: !Ref LambdaSecurityGroup
  VpcConfiguration:
    Description: VPC configuration summary for the Lambda function
    Value: !Sub 
      - "VPC: ${VpcId}, Subnets: ${SubnetList}, Security Groups: ${LambdaSecurityGroup}"
      - SubnetList: !Join [',', !Ref SubnetIds]</code></pre> 
<p>You need to run the following CLI command to deploy the CloudFormation stacks. You can change the ParameterValue of StackSetName“your-stackset-name” by the name of the StackSet you want to monitor. The default value is “security-baseline”. Your CLI profile should use region=“us-east-1“.</p> 
<p><code>aws cloudformation create-stack --stack-name stackset-monitor --template-body file://StackSetMonitor.yml --parameters ParameterKey=StackSetName,ParameterValue="security-baseline" --capabilities CAPABILITY_IAM</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to deploy the StackSetMonitor.yml CloudFormation template</em></span></p> 
<p>The CLI output should look like the following:</p> 
<p><code>{"StackId":&nbsp;"arn:aws:cloudformation:...."}</code></p> 
<p>Here’s the expected output for the CloudFormation template:</p> 
<p><img alt="StackSetMonitor Console output" class="size-full wp-image-23963 alignleft" height="607" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/7-StackSetMonitor-Console-Output.png" width="965" /></p> 
<p><span style="color: #808080;"><em>StackSetMonitor Console output</em></span></p> 
<p>And an example of Amazon CloudWatch Dashboard and Alarm screen:</p> 
<p><img alt="Amazon CloudWatch Dashboard screenshot for StackSetMonitor stack to track StackSet operations success rate" class="alignright size-full wp-image-23964" height="670" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/8-StackSetMonitor-CloudWatch-Dashboard.png" width="754" /></p> 
<p><em><span style="color: #808080;">Amazon CloudWatch Dashboard screenshot for StackSetMonitor stack to track StackSet operations success rate</span></em></p> 
<p><img alt="Amazon CloudWatch Alarm screenshot for StackSetMonitor stack to track StackSet operations success rate" class="size-full wp-image-23965 alignleft" height="470" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/9-StackSetMonitor-CloudWatch-Alarm.png" width="518" /><br /> <br style="clear: both;" /></p> 
<p><span style="color: #808080;"><em>Amazon CloudWatch Alarm screenshot for StackSetMonitor stack to track StackSet operations success rate</em></span></p> 
<p>SNS subscription setup involves retrieving the topic ARN from stack outputs and configuring notifications for email or SMS endpoints (below example CLI for email subscription):</p> 
<p><code>aws sns subscribe --topic-arn $SNS_TOPIC_ARN --protocol email --notification-endpoint your-email@example.com</code></p> 
<p><em><span style="color: #808080;">AWS CLI to subscribe to the topic providing the user email</span></em></p> 
<h2>Cost:</h2> 
<p>The estimated monthly expenses ranges between 5 and 15 USD depending on StackSet activity levels, with approximately 2,880 Lambda executions per day (each minute) under the default monitoring schedule.</p> 
<p>The solution supports customization of monitoring frequency by modifying the ScheduleExpression from the default one-minute interval. The cost will decrease if the monitoring is less frequent.</p> 
<h2>Cleanup:</h2> 
<p>For cleanup, you can run the following command lines:</p> 
<ul> 
 <li>To cleanup the Stack Instances and StackSets created in the Core Deployment Strategies section:</li> 
</ul> 
<p><code>aws cloudformation delete-stack-instances --stack-set-name security-baseline --deployment-targets OrganizationalUnitIds=ou-xxx --regions us-east-1 eu-west-1 --region us-east-1&nbsp;--no-retain-stack</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to delete the Stack Instances</em></span></p> 
<p>You need to change the parameter&nbsp;OrganizationalUnitIds value with the name of the OU, the parameter regions with the list of regions where you want to delete your stack instances, and the&nbsp;value of the stack-set-name parameter (security-baseline, monitoring-baseline, balanced-deployment…).</p> 
<p>Then you can delete the StackSet:</p> 
<p><code>aws cloudformation delete-stack-set&nbsp;--stack-set-name security-baseline</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to delete the StackSet</em></span></p> 
<p>You can change the value of the stack-set-name parameter.</p> 
<ul> 
 <li>To cleanup the stackset-monitor stack</li> 
</ul> 
<p><code>aws cloudformation delete-stack --stack-name stackset-monitor</code></p> 
<p><span style="color: #808080;"><em>AWS CLI to delete the stackset-monitor Stack</em></span></p> 
<p>You can also remove any IAM roles/policies that you specifically created for this blog that you might not need anymore</p> 
<h2>Conclusion</h2> 
<p>Throughout this guide, we’ve explored the nuanced approaches to AWS CloudFormation StackSets deployments across large-scale environments. The key takeaways include:</p> 
<ul> 
 <li><strong>Balance is Critical</strong>: Every deployment strategy requires careful consideration of the trade-offs between speed, safety, and scale based on your organizational needs.</li> 
 <li><strong>Progressive Adoption Works</strong>: For most organizations, a progressive deployment approach with validation gates provides the optimal balance of safety and efficiency.</li> 
 <li><strong>Organizational Context Matters</strong>: Enterprise, startup, and regulated industry patterns demonstrate that deployment strategies should be tailored to your specific business requirements and risk tolerance.</li> 
 <li><strong>Monitoring is Essential</strong>: As organizations scale to hundreds of accounts, comprehensive monitoring becomes critical for maintaining visibility and ensuring compliance.</li> 
</ul> 
<p>These different approaches will help you adopt the right strategy for your AWS CloudFormation Stacksets deployments in your AWS Organization.</p> 
<p>You can now test these different approaches on your sandbox environment, before adapting them for your specific needs, in order to balance Speed, Safety and Scale to optimize your deployments.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/07/PXL_20251007_134522658-1-scaled.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Amar Meriche</h3> 
  <p style="text-align: left;">Amar is a Sr Cloud Operations Architect at AWS in Paris. He helps his customers improve their operational posture through advocacy and guidance, and is an active member of the DevOps and IaC community at AWS. He’s passionate about helping customers use the various IaC tools available at AWS following best practices. When he’s not working with customers, Amar can be found on the mountain trails with his family or playing basketball with his team.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/08/idriss-profile-cut-scaled.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Idriss Laouali Abdou</h3> 
  <p style="text-align: left;">Idriss is a Sr. Product Manager Technical for AWS Infrastructure-as-Code based in Seattle. He focuses on improving developer productivity through StackSets and CloudFormation Infrastructure provisioning experiences. Outside of work, you can find him creating educational content for thousands of students, cooking, or dancing.</p> 
 </div> 
</footer>
