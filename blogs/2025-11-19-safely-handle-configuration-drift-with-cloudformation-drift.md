---
title: "Safely Handle Configuration Drift with CloudFormation Drift-Aware Change Sets"
url: "https://aws.amazon.com/blogs/devops/safely-handle-configuration-drift-with-cloudformation-drift-aware-change-sets/"
date: "Wed, 19 Nov 2025 18:54:15 +0000"
author: "JJ Lei"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<h2>Introduction</h2> 
<p>Is configuration drift preventing you from accessing the speed, safety, and governance benefits of <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> for infrastructure management? Configuration drift occurs when cloud resources are modified outside of CloudFormation, leading to a mismatch in the actual state and template definition of resources. Drift tends to accumulate from infrastructure changes that engineers make via the <a href="https://aws.amazon.com/console/">AWS Management Console</a> to resolve production incidents or troubleshoot malfunctioning applications. Drift can cause unexpected changes during subsequent <a href="https://aws.amazon.com/what-is/iac/">IaC</a> deployments or leave resources in a non-compliant state. Unresolved drift can lead to cost increases when resources are over-provisioned outside of template definitions, or compliance violations that may result in audit penalties. Additionally, drift makes it hard to reproduce applications for testing or disaster recovery.</p> 
<p>CloudFormation now offers <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/drift-aware-change-sets.html">drift-aware change sets</a> that allow you to safely handle configuration drift and keep your infrastructure in sync with your templates. In this post, we will explore the process of leveraging drift-aware change sets to resolve common scenarios in which drift impacts the availability or security of your application.</p> 
<h2>Solution Overview</h2> 
<p>Drift-aware change sets are a type of <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html">CloudFormation change sets</a> that can bring drifted resources in line with template definitions and preview the required changes to actual infrastructure states before deployment. Drift-aware change sets surface a three-way comparison of your new template, actual resource states, and previous template before deployment, allowing you to prevent unexpected overwrites of drift. Additionally, drift-aware change sets offer you a systematic mechanism to restore drifted resources to approved template definitions, strengthening the reproducibility and compliance posture of applications. You can create drift-aware change sets either from the CloudFormation Management Console or from the AWS CLI or SDK by passing the <code>--deployment-mode REVERT_DRIFT</code> parameter to the CreateChangeSet API.</p> 
<h2>Prerequisites</h2> 
<p>• <a href="https://aws.amazon.com/cli/">AWS CLI</a> latest version with CloudFormation permissions configured.</p> 
<p>• <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management</a> (IAM) permissions required: Permissions to create and manage CloudFormation stacks, <a href="https://aws.amazon.com/pm/lambda/">AWS Lambda</a> functions, Security Groups, <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service</a> (Amazon S3) buckets, and IAM roles. PowerUserAccess or Administrator access recommended for testing.</p> 
<p>• Test environment (non-production AWS account recommended)</p> 
<p>• Basic CloudFormation knowledge (stacks, templates, change sets)</p> 
<p><strong>Important Note</strong>: These sample templates are provided for educational purposes only and should not be used in production environments without proper security review and testing. You are responsible for testing, securing, and optimizing these templates based on your specific quality control practices and standards. Deploying these templates may incur AWS charges for creating or using AWS resources. Work with your security and legal teams to meet your organizational security, regulatory, and compliance requirements before any production deployment.</p> 
<h2>Scenario 1: Prevent Dangerous Overwrites</h2> 
<p>This scenario demonstrates how drift-aware change sets prevent dangerous overwrites when Lambda function memory is increased outside of CloudFormation during an outage, and a subsequent template update could accidentally reduce memory, causing performance issues.</p> 
<p><strong>Story</strong>: Your team deploys a Lambda function with 128 MB memory via CloudFormation. During a production outage, an engineer increases the memory to 512 MB through the Lambda Console to resolve performance issues. Later, another developer updates the template to 256 MB for a code change, unaware of the console modification. Without drift-aware change sets, CloudFormation would unexpectedly reduce memory from 512 MB to 256 MB—potentially causing the outage to recur.</p> 
<p><strong>User journey</strong>: Create stack with 128MB =&gt; Increase memory to 512MB via console during outage =&gt; Create drift-aware change set with 256MB template =&gt; Review three-way comparison showing dangerous memory reduction =&gt; Cancel change set to prevent outage =&gt; Update template to match production state (512MB) =&gt; Create and execute drift-aware change set with updated template (512MB) to resolve drift</p> 
<h3>Scenario Flow</h3> 
<h4>1. Create Stack</h4> 
<p>Deploy CloudFormation stack with Lambda function (128 MB memory).</p> 
<div class="wp-caption aligncenter" id="attachment_24322" style="width: 1034px;">
 <img alt="Figure 1" class="wp-image-24322 size-large" height="183" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure1-stack-deployed-1-1024x183.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24322">CloudFormation stack “lambda-memory-drift-test” successfully deployed with CREATE_COMPLETE status</p>
</div> 
<h4>2. Emergency Memory Increase (Console)</h4> 
<p>Manually increase Lambda memory to 512 MB through AWS Console (simulating emergency performance fix during outage).</p> 
<div class="wp-caption aligncenter" id="attachment_24323" style="width: 1034px;">
 <img alt="Figure 2" class="wp-image-24323 size-large" height="182" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure2-lambda-128mb-1-1024x182.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24323">Initial Lambda function showing 128 MB memory as configured in template</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_24324" style="width: 1034px;">
 <img alt="Figure 3" class="wp-image-24324 size-large" height="178" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure3-lambda-512mb-1-1024x178.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24324">Lambda memory increased to 512 MB through console during outage, creating drift from template</p>
</div> 
<h4>3. Create Drift-Aware Change Set</h4> 
<p>Create change set with 256 MB template using drift-aware mode to reveal the dangerous memory reduction.</p> 
<div class="wp-caption aligncenter" id="attachment_24325" style="width: 1034px;">
 <img alt="Figure 4" class="wp-image-24325 size-large" height="761" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure4-drift-aware-option-1-1024x761.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24325">CloudFormation console showing the new “Drift aware change set” option selected. This compares the new template with the live state of your stack and shows changes to drifted resources before deployment, unlike standard change sets that only compare templates.</p>
</div> 
<pre><code class="language-bash">aws cloudformation create-change-set \
--stack-name lambda-memory-drift-test \
--change-set-name detect-memory-overwrite \
--template-body file://lambda-memory-drift-scenario-256mb.yaml \
--deployment-mode REVERT_DRIFT \
--capabilities CAPABILITY_IAM \
--region us-east-1
</code></pre> 
<h4>4. Review Change Set – The Critical Three-Way Comparison</h4> 
<p>Examine the drift-aware change set to see the dangerous memory reduction that would occur.</p> 
<div class="wp-caption aligncenter" id="attachment_24387" style="width: 1034px;">
 <img alt="Figure 5" class="wp-image-24387 size-large" height="762" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure5-memory-reduction-1-2-1024x762.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24387">Critical insight revealed: The change set shows Live resource state (512 MB) vs Proposed resource state (256 MB), revealing a dangerous memory reduction that would impact performance.</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_24388" style="width: 1034px;">
 <img alt="Figure 6: view drift" class="wp-image-24388 size-large" height="749" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure6-view-drift-1-2-1024x749.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24388">Drift analysis: Clicking “View drift” reveals the complete picture – Previous template (128 MB) vs Live resource state (512 MB). This shows the live state has 4x more memory than the original template, indicating emergency changes were made during the outage that must be preserved.</p>
</div> 
<p><strong>Key Insight</strong>: The drift-aware change set reveals that:</p> 
<ul> 
 <li><strong>Previous template</strong>: 128 MB (original deployment)</li> 
 <li><strong>Live resource state</strong>: 512 MB (emergency change during outage)</li> 
 <li><strong>Proposed template</strong>: 256 MB (new deployment)</li> 
</ul> 
<p>This would cause a <strong>dangerous reduction</strong> from 512 MB to 256 MB, potentially recreating the original performance issue. Without drift-aware change sets, this critical information would be hidden.</p> 
<h4>5. Recreate Drift-aware Change Set with Updated Template (512MB) to Resolve Drift</h4> 
<p>Update the template to match the live production state (512 MB) and create a new drift-aware change set to safely resolve the drift.</p> 
<div class="wp-caption aligncenter" id="attachment_24368" style="width: 1034px;">
 <img alt="Figure 7" class="wp-image-24368 size-large" height="475" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure7-resolution-confirmed-2-1024x475.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24368">Resolution confirmed: The drift-aware change set shows both Live resource state and Proposed resource state at 512 MB, with change set action “Sync with live”. This verifies that the updated template now matches production, preventing the dangerous memory reduction and safely resolving the drift without impacting performance.</p>
</div> 
<h3>CloudFormation Templates</h3> 
<p><strong>Initial Template (128 MB – </strong><code>lambda-memory-drift-scenario.yaml</code><strong>):</strong></p> 
<pre><code class="language-yaml">Resources:
  DriftTestFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.lambda_handler
      MemorySize: 128
      ReservedConcurrentExecutions: 5
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          def lambda_handler(event, context):
              return {'statusCode': 200, 'body': 'Hello!'}
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
</code></pre> 
<p><strong>Updated Template (256 MB –</strong> <code>lambda-memory-drift-scenario-256mb.yaml</code><strong>):</strong></p> 
<pre><code class="language-yaml">Resources:
  DriftTestFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.lambda_handler
      MemorySize: 256
      ReservedConcurrentExecutions: 5
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          def lambda_handler(event, context):
              return {'statusCode': 200, 'body': 'Hello!'}
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
</code></pre> 
<h3>CLI Commands</h3> 
<ol> 
 <li><strong>Create stack:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-stack --stack-name lambda-memory-drift-test --template-body file://lambda-memory-drift-scenario.yaml --capabilities CAPABILITY_IAM --region us-east-1
</code></pre> 
<ol start="2"> 
 <li><strong>Get function name:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation describe-stack-resources --stack-name lambda-memory-drift-test --logical-resource-id DriftTestFunction --query 'StackResources[0].PhysicalResourceId' --output text --region us-east-1
</code></pre> 
<ol start="3"> 
 <li><strong>Create drift-aware change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-change-set --stack-name lambda-memory-drift-test --change-set-name detect-memory-overwrite --template-body file://lambda-memory-drift-scenario-256mb.yaml --deployment-mode REVERT_DRIFT --capabilities CAPABILITY_IAM --region us-east-1
</code></pre> 
<ol start="4"> 
 <li><strong>Describe change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation describe-change-set --change-set-name detect-memory-overwrite --stack-name lambda-memory-drift-test --region us-east-1
</code></pre> 
<h2>Scenario 2: Remediate Unauthorized Changes</h2> 
<p>This scenario demonstrates how drift-aware change sets systematically remediate unauthorized changes when a developer adds temporary debugging rules to a security group but forgets to remove them, creating a compliance violation.</p> 
<p><strong>Story</strong>: Your team deploys a security group with only HTTP access via CloudFormation for compliance. During debugging, a developer adds SSH access (port 22) through the AWS Console for their IP address to troubleshoot an application issue. They forget to remove this rule after debugging. Later, security compliance requires reverting to the original template state. A standard change set shows no changes since the template is unchanged, but a drift-aware change set can detect and systematically remove the unauthorized SSH rule.</p> 
<p><strong>User journey</strong>: Create stack with HTTP-only access =&gt; Add SSH rule via console for debugging =&gt; Forget to remove SSH rule =&gt; Create drift-aware change set with REVERT_DRIFT mode =&gt; Review change set showing SSH rule removal =&gt; Execute change set to restore compliance</p> 
<h3>Scenario Flow</h3> 
<h4>1. Create Stack</h4> 
<p>Deploy CloudFormation stack with security group allowing only HTTP traffic.</p> 
<div class="wp-caption aligncenter" id="attachment_24330" style="width: 1034px;">
 <img alt="Figure 8" class="wp-image-24330 size-large" height="296" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure8-sg-stack-deployed-1-1024x296.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24330">CloudFormation stack “sg-revert-drift-test” successfully deployed with DriftTestSecurityGroup resource</p>
</div> 
<h4>2. Make Unauthorized Changes (Console)</h4> 
<p>Manually add SSH ingress rule through AWS Console (simulating developer debugging access that wasn’t removed).</p> 
<div class="wp-caption aligncenter" id="attachment_24369" style="width: 1034px;">
 <img alt="Figure 9: http only" class="wp-image-24369 size-large" height="483" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure9-http-only-1-1-1024x483.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24369">Initial security group showing only HTTP (port 80) access as configured in template – compliant state</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_24370" style="width: 1034px;">
 <img alt="Figure 10: ssh-added" class="wp-image-24370 size-large" height="509" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure10-ssh-added-1-1024x509.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24370">Security group now shows 2 permission entries: SSH (port 22) for specific IP and HTTP (port 80) for all traffic. The SSH rule creates drift and a compliance violation that needs systematic removal.</p>
</div> 
<h4>3. Create Drift-Aware Change Set</h4> 
<p>Create change set using REVERT_DRIFT mode to systematically remove the unauthorized SSH rule.</p> 
<div class="wp-caption aligncenter" id="attachment_24306" style="width: 1034px;">
 <img alt="Figure 11" class="wp-image-24306 size-large" height="527" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure11-sg-changeset-creation-1024x527.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24306">Creating drift-aware change set for security group compliance restoration. Note the “Drift aware change set” option is selected to compare with live state and detect unauthorized changes.</p>
</div> 
<pre><code class="language-bash">aws cloudformation create-change-set \
--stack-name sg-revert-drift-test \
--change-set-name revert-ssh-drift \
--use-previous-template \
--deployment-mode REVERT_DRIFT \
--region us-east-1
</code></pre> 
<h4>4. Review Change Set – Systematic Compliance Restoration</h4> 
<p>Examine the drift-aware change set to see systematic removal of unauthorized SSH rule.</p> 
<div class="wp-caption aligncenter" id="attachment_24389" style="width: 1034px;">
 <img alt="Figure 12" class="wp-image-24389 size-large" height="905" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/scenario2_05_changeset_creation_review_Resources_Changets_Live_resources_state_on_the_left_and_Proposed_resource_state_on_the_rightthree_way_comparison-1024x905.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24389">Compliance violation detected: The drift -aware change set shows that the SSH rule in the live resource state (rule 232 for IP 15.248.7.53/32 on port 22) is not present in the proposed resource state derived from the template. This unauthorized SSH rule violates security policy and will be systematically removed</p>
</div> 
<p><strong>Key Insight</strong>: The drift-aware change set enables systematic compliance restoration by:</p> 
<ul> 
 <li><strong>Previous template</strong>: Only HTTP (port 80) access – compliant state</li> 
 <li><strong>Live resource state</strong>: HTTP + SSH (port 22) for 15.248.7.53/32 – compliance violation</li> 
 <li><strong>Action</strong>: Remove unauthorized SSH rule to restore compliance</li> 
</ul> 
<p>This provides a systematic, auditable way to remove unauthorized changes rather than manual cleanup.</p> 
<div class="wp-caption aligncenter" id="attachment_24373" style="width: 1034px;">
 <img alt="Figure 13" class="wp-image-24373 size-large" height="650" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure13-ssh-removed-1-1024x650.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24373">Stack events showing successful execution of the drift-aware change set – SSH rule removed</p>
</div> 
<h3>CloudFormation Templates</h3> 
<p><code>security-group-drift-scenario.yaml</code><strong>:</strong></p> 
<pre><code class="language-yaml">Resources:
  DriftTestSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Security group for drift testing"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: "Allow HTTP traffic for demo purposes"
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
          Description: "Allow all outbound traffic"
</code></pre> 
<h3>CLI Commands</h3> 
<ol> 
 <li><strong>Create stack:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-stack --stack-name sg-revert-drift-test --template-body file://security-group-drift-scenario.yaml --region us-east-1
</code></pre> 
<ol start="2"> 
 <li><strong>Get security group ID:</strong></li> 
</ol> 
<pre><code class="language-bash">aws ec2 describe-security-groups --filters "Name=tag:aws:cloudformation:stack-name,Values=sg-revert-drift-test" --query 'SecurityGroups[0].GroupId' --output text --region us-east-1
</code></pre> 
<ol start="3"> 
 <li><strong>Create drift-aware change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-change-set --stack-name sg-revert-drift-test --change-set-name revert-ssh-drift --template-body file://security-group-drift-scenario.yaml --deployment-mode REVERT_DRIFT --region us-east-1
</code></pre> 
<ol start="4"> 
 <li><strong>Describe change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation describe-change-set --change-set-name revert-ssh-drift --stack-name sg-revert-drift-test --region us-east-1
</code></pre> 
<h2>Scenario 3: Recreate Deleted Resources</h2> 
<p>This scenario demonstrates drift detection when a dependent resource (logs bucket) is accidentally deleted outside of CloudFormation during troubleshooting. The main application bucket depends on this logs bucket for access logging. You need to recreate the deleted resource while maintaining the existing infrastructure dependencies.</p> 
<p><strong>Story</strong>: Your team deploys a main S3 bucket with a dependent logs bucket for access logging via CloudFormation. During troubleshooting, an operator accidentally deletes the logs bucket through the AWS Console. The main bucket still exists but its logging configuration now references a non-existent bucket. You need to recreate the deleted logs bucket while maintaining the dependency relationship.</p> 
<p><strong>User journey</strong>: Create stack with main and logs buckets =&gt; Accidentally delete logs bucket =&gt; Create drift-aware change set with REVERT_DRIFT mode =&gt; Review change set showing LogBucket will be recreated =&gt; Execute change set to restore deleted resource</p> 
<h3>Scenario Flow</h3> 
<h4>1. Create Stack</h4> 
<p>Deploy CloudFormation stack with main S3 bucket and dependent logs bucket.</p> 
<div class="wp-caption aligncenter" id="attachment_24303" style="width: 1034px;">
 <img alt="Figure 14" class="wp-image-24303 size-large" height="310" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure14-s3-stack-deployed-1024x310.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24303">CloudFormation stack “s3-deletion-drift-test” successfully deployed with both LogBucket and MainBucket resources in CREATE_COMPLETE status</p>
</div> 
<h4>2. Accidental Deletion (Console)</h4> 
<p>Manually delete the logs bucket through AWS Console (simulating accidental deletion during troubleshooting).</p> 
<div class="wp-caption aligncenter" id="attachment_24302" style="width: 1034px;">
 <img alt="Figure 15" class="wp-image-24302 size-large" height="213" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure15-bucket-deleted-1024x213.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24302">LogBucket accidentally deleted outside of CloudFormation during troubleshooting, creating drift – the MainBucket still exists but its logging configuration now references a non-existent bucket</p>
</div> 
<h4>3. Create Drift-Aware Change Set</h4> 
<p>Create change set using REVERT_DRIFT mode to recreate the deleted LogBucket.</p> 
<div class="wp-caption aligncenter" id="attachment_24301" style="width: 1034px;">
 <img alt="Figure 16" class="wp-image-24301 size-large" height="407" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/18/figure16-s3-changeset-creation-1024x407.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24301">Creating drift-aware change set with “Drift aware change set” option selected to detect and recreate the deleted resource by comparing template with live state</p>
</div> 
<pre><code class="language-bash">aws cloudformation create-change-set \
--stack-name s3-deletion-drift-test \
--change-set-name recreate-deleted-bucket \
--use-previous-template \
--deployment-mode REVERT_DRIFT \
--region us-east-1
</code></pre> 
<h4>4. Review Change Set – Resource Recreation</h4> 
<p>Examine change set to see LogBucket recreation while preserving MainBucket dependencies.</p> 
<div class="wp-caption aligncenter" id="attachment_24375" style="width: 1034px;">
 <img alt="Figure 17" class="wp-image-24375 size-large" height="638" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/figure17-bucket-recreation-1-1024x638.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-24375">Change set preview showing LogBucket will be recreated to restore the deleted resource and MainBucket updated to maintain infrastructure dependencies</p>
</div> 
<p><strong>Key Insight</strong>: The drift-aware change set detects that:</p> 
<ul> 
 <li><strong>Template expectation</strong>: Both LogBucket and MainBucket should exist</li> 
 <li><strong>Live resource state</strong>: Only MainBucket exists, LogBucket is missing</li> 
 <li><strong>Action</strong>: Recreate LogBucket with original configuration to restore logging functionality</li> 
</ul> 
<p>This enables systematic recovery of accidentally deleted resources while maintaining infrastructure dependencies.</p> 
<h3>CloudFormation Templates</h3> 
<p><code>s3-drift-scenario.yaml</code><strong>:</strong></p> 
<pre><code class="language-yaml">Resources:
  LogBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      VersioningConfiguration:
        Status: Enabled
  
  MainBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      VersioningConfiguration:
        Status: Enabled
      LoggingConfiguration:
        DestinationBucketName: !Ref LogBucket
</code></pre> 
<h3>CLI Commands</h3> 
<ol> 
 <li><strong>Create stack:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-stack --stack-name s3-deletion-drift-test --template-body file://s3-drift-scenario.yaml --region us-east-1
</code></pre> 
<ol start="2"> 
 <li><strong>Get LogBucket name:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation describe-stack-resources --stack-name s3-deletion-drift-test --logical-resource-id LogBucket --query 'StackResources[0].PhysicalResourceId' --output text --region us-east-1
</code></pre> 
<ol start="3"> 
 <li><strong>Create drift-aware change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation create-change-set --stack-name s3-deletion-drift-test --change-set-name recreate-deleted-bucket --template-body file://s3-drift-scenario.yaml --deployment-mode REVERT_DRIFT --region us-east-1
</code></pre> 
<ol start="4"> 
 <li><strong>Describe change set:</strong></li> 
</ol> 
<pre><code class="language-bash">aws cloudformation describe-change-set --change-set-name recreate-deleted-bucket --stack-name s3-deletion-drift-test --region us-east-1
</code></pre> 
<h2>Best Practices</h2> 
<p>When working with drift-aware change sets, consider these best practices:</p> 
<p>• <strong>Always review three-way comparisons</strong> before executing change sets to understand the full impact</p> 
<p>• <strong>Use REVERT_DRIFT deployment mode</strong> when you want to bring resources back to template compliance</p> 
<p>• <strong>Document emergency changes</strong> made outside of CloudFormation to inform future template updates</p> 
<p>• <strong>Implement change management processes</strong> to minimize unauthorized drift</p> 
<p>• <strong>Regular drift detection</strong> helps identify configuration changes before they become problematic</p> 
<p>• <strong>Test drift-aware change sets</strong> in non-production environments first</p> 
<h2>Cleanup</h2> 
<p><strong>Important:</strong> Execute these cleanup commands promptly after completing the scenarios to avoid incurring unnecessary AWS charges. Resources such as Lambda functions, S3 buckets (even if empty), and security groups may incur costs if left running. Ensure all stacks are successfully deleted by verifying the DELETE_COMPLETE status.</p> 
<p><strong>Commands to delete all test resources:</strong></p> 
<pre><code class="language-bash"># Scenario 1: Lambda Memory Drift
aws cloudformation delete-stack --stack-name lambda-memory-drift-test --region us-east-1

# Scenario 2: Security Group Drift
aws cloudformation delete-stack --stack-name sg-revert-drift-test --region us-east-1

# Scenario 3: S3 Bucket Deletion Drift
aws cloudformation delete-stack --stack-name s3-deletion-drift-test --region us-east-1

# Verify all stacks are deleted
aws cloudformation list-stacks --stack-status-filter DELETE_COMPLETE --region us-east-1
</code></pre> 
<p><strong>Note:</strong> CloudFormation will automatically clean up all resources created by the stacks, including Lambda functions, security groups, and S3 buckets.</p> 
<h2>Conclusion</h2> 
<p>Drift-aware change sets enable you to mitigate the operational and security risks of configuration drift, allowing you to confidently automate and govern your infrastructure updates with CloudFormation. Through the scenarios described in this post, you have seen how you can leverage drift-aware change sets to prevent outages in production environments, maintain the integrity of your test environments, and manage the compliance posture of all environments. Remember to thoroughly review the infrastructure changes previewed by drift-aware change sets before executing deployments.</p> 
<h2>Available Now</h2> 
<p>Drift-aware change sets are available in AWS Regions where CloudFormation is available. Please refer to the <a href="https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/">AWS Region table</a> to learn more.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/jjlei-3.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">JJ Lei</h3> 
  <p style="text-align: left;">JJ Lei is a Solutions Architect with the AWS Consulting Partner SA team, where he helps partners develop cloud practices with a focus on Infrastructure as Code, including CloudFormation, CDK, and DevOps/Developer tooling. Outside of work, he enjoys long walks in parks with an unnecessarily large backpack.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/11/19/IMG_2720-1.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Aakash Deshpande</h3> 
  <p style="text-align: left;">Aakash Deshpande is a Senior Product Manager for AWS CloudFormation, where he drives the vision for infrastructure provisioning and management experiences on AWS.</p> 
 </div> 
</footer>
