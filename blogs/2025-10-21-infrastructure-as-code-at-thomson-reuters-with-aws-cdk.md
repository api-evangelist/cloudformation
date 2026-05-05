---
title: "Infrastructure as Code at Thomson Reuters with AWS CDK"
url: "https://aws.amazon.com/blogs/devops/infrastructure-as-code-at-thomson-reuters-with-aws-cdk/"
date: "Tue, 21 Oct 2025 07:28:37 +0000"
author: "Vu San Ha Huynh"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<p><em>This post is cowritten by Danilo Tommasina and Lalit Kumar B from Thomson Reuters.</em></p> 
<p>Large organizations often struggle with infrastructure management challenges including compliance issues, development bottlenecks and errors from inconsistent AWS resource creation across teams. Without standardized naming, tagging and policy enforcement, teams face repeated boilerplate code and difficulty accessing centrally-managed resources.</p> 
<p>In this post, we will show you how <a href="https://www.thomsonreuters.com/en.html">Thomson Reuters</a>&nbsp;developed an extension of the <a href="https://aws.amazon.com/cdk/">AWS Cloud Development Kit (CDK)</a> to automate compliance, standardization and policy enforcement in Infrastructure as Code (IaC) scripts. We will explore the strategic reasoning behind this initiative, outline foundational design principles, and provide technical details on TR’s journey from concept to implementation. The solution accelerates and standardizes cloud infrastructure deployment and management through seamless integration between TR’s custom library and AWS CDK.</p> 
<p>Thomson Reuters (TR) is one of the world’s leading information organizations for businesses and professionals. TR provides companies with the intelligence, technology, and human expertise they need to find trusted answers, enabling them to make better decisions more quickly. TR’s customers span the financial, risk, legal, tax, accounting, and media industries.</p> 
<h2>Overview</h2> 
<p>In a large organization that offers a variety of customer products, it is essential to manage numerous cloud resources effectively. This involves overseeing multiple AWS accounts, implementing access control or addressing financial tracking challenges. These tasks require the application of centrally defined standards and conventions, with additional requirements tailored to specific sub-organizations.</p> 
<p>Infrastructure as Code (IaC) is an effective method for managing cloud resources. However, utilizing vanilla <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> for extensive and intricate infrastructure can pose challenges. It requires careful attention to naming conventions, tagging standards, security, and best practices for infrastructure deployments. Additionally, repeating infrastructure patterns across various services and products often leads to excessive use of copy-paste and dealing with boilerplate code. When projects require configurable and dynamic components – including conditionals, loops, repeatable patterns, and distribution to a large user base – delivering CloudFormation scripts can become quite cumbersome and prone to errors.</p> 
<p>AWS CDK addresses these challenges by enabling IaC development in high-level programming languages like TypeScript, JavaScript, Python, Java. <a href="https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs-lib-levels">AWS CDK Level 2 and 3</a> constructs simplify and reduce the amount of code to be written to manage complex infrastructure. It allows TR to create custom libraries that extend the vanilla AWS CDK with additional patterns and utilities. The extension libraries can also be distributed for multiple programming languages and package managers thanks to <a href="https://aws.github.io/jsii/">JSII</a>. JSII enables TypeScript libraries to be automatically compiled and packaged for native consumption in each target language, allowing CDK libraries to be written once but used in many different programming environments.</p> 
<h2>Solution to optimize the process</h2> 
<p>In a medium to large company, different teams provide the fundamental infrastructure services (e.g. authentication and authorization, networking, security, financial tracking and optimization, base infrastructure provisioning, etc.) to enable use of the cloud for a large community of developers.</p> 
<p>Figure 1 illustrates the conventional method involving teams producing documentation that outlines the usage of pre-deployed infrastructure. This includes naming and tagging standards, required security boundaries, default settings and other relevant guidelines. Subsequently, the implementation team reviews these documents and integrates the established rules into their tool chain consistently, often working in isolation. This results in inefficiencies, misinterpretation risks and maintenance challenges when specifications change.</p> 
<p><img alt="Figure 1. The traditional approach with separate documentation and implementation teams." class="size-full wp-image-24068 aligncenter" height="668" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/Figure-1-2.png" width="1006" /></p> 
<p><em>Figure 1: The traditional approach</em></p> 
<p>TR’s optimized approach replaces documentation with working code as shown in Figure 2.</p> 
<p><img alt="Figure 2: The optimized approach with shared CDK extension library" class="size-full wp-image-24069 aligncenter" height="849" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/Figure-2-1.png" width="1384" /></p> 
<p><em>Figure 2: The optimized approach</em></p> 
<p>Infrastructure teams contribute their specifications into an extension library for AWS CDK, while the implementation teams can also contribute common patterns back into the central extension. The central extension library is released as polyglot packages allowing the implementation teams to pick the programming language that fits best to their knowledge.</p> 
<p>With this approach, TR introduce a “shift-left” in the development and delivery lifecycle. Standards and best practices are introduced early, things are done right by default, and TR minimizes the risks of getting inappropriately configured resources to be deployed, which leads to a reduction in the number of governance and security incidents.<br /> Implementation delivery teams can share well architected patterns for re-use by other teams to improve overall effectiveness.</p> 
<h2>Implementation</h2> 
<h3>Design principles</h3> 
<p>Key factors for the adoption of a framework are:</p> 
<ul> 
 <li>Simplicity, ease-of-use, self-service, and fast onboarding</li> 
 <li>Low maintenance effort and cost</li> 
 <li>Controlled roll-out, ability to quickly roll-back</li> 
</ul> 
<p>With the above in mind, TR delivered a minimally invasive framework that can be enabled with a tiny set of custom code on top of vanilla AWS CDK code.</p> 
<p>Using the TR-AWS CDK core library is straightforward – users simply import the package and adapt their entry point. From there, they can leverage standard AWS CDK code and documentation for most development tasks. There’s no need to learn custom construct classes or follow extensive specialized tutorials – vanilla AWS CDK knowledge is sufficient for most requirements. Additionally, developers can quickly incorporate open-source construct libraries through standard package managers. These third-party libraries integrate seamlessly with the TR implementation, automatically conforming to company standards without requiring additional configuration.</p> 
<p>By managing distribution of the library following standard software packaging and release procedures TR enable consumers to adopt new capabilities in a controlled way, with the ability to roll-back to previous versions if something goes wrong during an update.</p> 
<p>All this together allows TR to tick off the key factors listed above.</p> 
<h3>The monorepo approach</h3> 
<p>TR created a monorepo (monolithic repository) which is a version control strategy where multiple projects or packages are stored in a single repository. This approach offers several advantages over maintaining separate repositories for each package: unified versioning, simplified dependency management, consistent tooling, atomic changes across packages and improved collaboration.</p> 
<p>This setup mirrors the configuration used by AWS CDK itself.</p> 
<p>TR organized their monorepo following this structure:</p> 
<ul> 
 <li><code>repo/package.json</code>: Defines dev dependencies and global scripts used by all packages</li> 
 <li><code>repo/packages</code>: contains the different modules</li> 
 <li><code>repo/packages/core/package.json</code>: deps of core module and scripts for core module</li> 
 <li><code>repo/packages/core/lib/*</code>: typescript code that composes the core module</li> 
 <li><code>repo/packages/core/lib/augmentation/*</code>: module augmentations for AWS CDK core components</li> 
 <li><code>repo/packages/constructs-pattern-X</code>: define multiple reusable and independent level 3 constructs</li> 
 <li><code>repo/packages/tr-cdk-lib/package.json</code>: assembly module that defines scripts to assemble the final mono package that will be shared via a npm repository</li> 
</ul> 
<p><img alt="Figure 3. The monorepo structure" class="size-full wp-image-24070 aligncenter" height="562" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/Figure-3-1.png" width="306" /></p> 
<p><em>Figure 3: Repo structure</em></p> 
<p>This structure enables TR to maintain a collection of related, but distinct CDK constructs while making sure they work together seamlessly.</p> 
<p>The modules are assembled and released into one single versioned package which simplifies the end-user’s consumption.</p> 
<h3>The core module: Foundation of TR AWS CDK library</h3> 
<p>The core module is the foundation of TR’s CDK extension library, it consists of several key components that work together to “TR-ify” AWS resources and offer simplified access to centrally managed infrastructure resources that are provided by TR’s <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/migration-aws-environment/understanding-landing-zones.html">AWS landing zone</a> teams.</p> 
<p>TR refers to “TR-ification”, as the process of dynamically adapting AWS CDK constructs to meet their standards and best practices. From a user perspective, the process happens in a minimally invasive way, for most of the time the user is coding with vanilla AWS CDK components, while having access to short-cuts to a variety of TR specific resources.</p> 
<p>The core module serves several critical purposes:</p> 
<ol> 
 <li><strong>Standardization</strong>: makes sure the AWS resources follow TR naming conventions and tagging standards</li> 
 <li><strong>Simplification</strong>: abstracts away complex configurations required for TR compliance</li> 
 <li><strong>Integration</strong>: provides seamless access to TR-managed resources like VPCs, security groups, and Route53 hosted zones</li> 
 <li><strong>Policy Enforcement</strong>: automatically applies custom security and financial optimization policies</li> 
</ol> 
<p>The “TR-ification” process happens on every construct following a consistent order, for each construct it will:</p> 
<ol> 
 <li>If applicable, set a name following a consistent pattern</li> 
 <li>Apply custom initialization logic (e.g. set IAM permission boundary)</li> 
 <li>Apply security and financial optimization defaults (if not set)</li> 
 <li>Perform custom validations</li> 
 <li>Verify security and financial optimization policies</li> 
 <li>Tag resources</li> 
</ol> 
<p>TR uses a single root-level <a href="https://docs.aws.amazon.com/cdk/v2/guide/aspects.html">Aspect</a> instead of multiple Aspects to avoid complex resource type checking and improve maintainability:</p> 
<pre><code class="lang-ts">// This is the entrypoint that triggers the trification process on all CDK constructs
// we apply all TR specific transformations at this point
Aspects.of(this).add({
  visit: (node: IConstruct) =&gt; {
    node.getTRifier().trify();
  },
});</code></pre> 
<p>The careful readers at this point will scream:<br /> Wait a moment! <code>node.getTRifier().trify()</code> won’t compile!</p> 
<p>Which is absolutely correct… unless you know a topic in TypeScript called <a href="https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation">module augmentation</a>, in TR’s case, they augment the <code>IConstruct</code> interface and <code>Construct</code> class as follows:</p> 
<pre><code class="lang-ts">/** Defines the set of functionality needed when trifying resources */
export interface ITRifier {
    trify(): void;
    readonly name: string | undefined;
    readonly nameFromTree: string;
}

declare module 'constructs/lib/construct' {
    interface IConstruct {
        /** Obtain the ITRifier responsible to add TR specific features to this CDK IConstruct */
        getTRifier(): ITRifier;
        
        trContext(): AppContext | StageContext | StackContext;
    }
    
    interface Construct extends IConstruct {
        /** Build the ITRifier responsible to add TR specific features to this CDK IConstruct */
        buildTRifier(): ITRifier;
    }
}</code></pre> 
<p>Then provide default implementations for the generic Construct:</p> 
<pre><code class="lang-ts">Construct.prototype.getTRifier = function () {
    // Lazy getter, build the TRifier only when needed and cache it
    return ObjectUtils.lazyGetFrom(this, 'trifier', () =&gt; this.buildTRifier());
};

Construct.prototype.buildTRifier = function () {
    return new ConstructTRifier(this); // Default dummy implementation
};

Construct.prototype.trContext = function (): StackContext {
    return Stack.of(this).trContext() as StackContext;
};</code></pre> 
<p>Since AWS CDK constructs implement the <code>IConstruct</code> interface, respectively extend the <code>Construct</code> class automatically, the “TR-ification” process becomes available for many types of constructs.<br /> All you need to do now is inject your custom logic for all resources you need customization and make sure the module is loaded, e.g. in case of a Lambda function, it uses:</p> 
<pre><code class="lang-ts">lambda.CfnFunction.prototype.buildTRifier = function () {
    return new CfnResourceTRifierLambda.CfnFunction(
        this,
        () =&gt; { // Accessor for retrieving the lambda function name
            return this.functionName;
        },
        (name: string) =&gt; { // Accessor for setting the lambda function name
                this.functionName = name;
        },
        () =&gt; {
            // Our own stuff to set defaults for financial optimizations
            const policyChecker = FinOps.Lambda.Defaults.apply(this);
            
            this.node.addValidation({
                validate: () =&gt; {
                    // Inject a custom validation logic to check compliance with financial policies
                    return policyChecker.addErrorIfNotCompliant(this);
                }
            });
        }
    );
};</code></pre> 
<p>TR targets L1 (Cfn) constructs like CfnFunction because the higher-level L2 and L3 constructs internally create L1 constructs during synthesis. This architectural decision makes sure TR-ification is applied universally, whether users write new lambda.Function() or new lambda.CfnFunction(), both will be TR-ified. This approach provides complete coverage with a single implementation point while remaining completely transparent to library users who can continue using their preferred abstraction level without awareness of this internal mechanism.</p> 
<h3>Naming standardization</h3> 
<p>TR uses standardized naming to support IAM policy filtering and consistent resource management. In order to support a broad range of use-cases, TR defined the resource name pattern as follows:<br /> <code>&lt;segregationPrefix&gt;[-appPrefix]-&lt;resourceName&gt;[-region]-&lt;envSuffix&gt;</code><br /> where the elements mean:</p> 
<ul> 
 <li><code>segregationPrefix</code>: A prefix used for grouping resources for a specific asset, it implies that a segregated administrative group is responsible for this resource, where applicable it is used for ARN based IAM resource filtering.</li> 
 <li><code>appPrefix</code>: Optional, a prefix used to map a resource to a specific application or service, this is shared across stacks within a <a href="https://docs.aws.amazon.com/cdk/v2/guide/apps.html">CDK app</a>.</li> 
 <li><code>resourceName</code>: The name of a resource indicating its purpose.</li> 
 <li><code>region</code>: Optional, applied only to resources that are global but are part of a <a href="https://docs.aws.amazon.com/cdk/v2/guide/stacks.html">CDK stack</a> that is bound to a specific region.</li> 
 <li><code>envSuffix</code>: A suffix used to segregate different deployment environments, e.g. development, continuous integration, quality assurance, production.</li> 
</ul> 
<p>Traditional approaches require developers to manually construct these names, propagating prefixes and suffixes throughout their code:</p> 
<pre><code class="lang-ts">new lambda.Function(stack, 'foo', {
    runtime: lambda.Runtime.NODEJS_LATEST,
    handler: 'index.handler',
    code: new lambda.InlineCode('bar'),
    functionName: `\${segregationPrefix}-\${appPrefix}-compute-stats-\${envSuffix}`,
});</code></pre> 
<p>With TR AWS CDK extension, the code is simplified to:</p> 
<pre><code class="lang-ts">new lambda.Function(stack, 'MyFunction', {
  runtime: lambda.Runtime.NODEJS_LATEST,
  handler: 'index.handler',
  code: new lambda.InlineCode('foo'),
  functionName: 'compute-stats',
});</code></pre> 
<p>The <code>functionName</code> describes what the function does without “noise”, TR AWS CDK will transparently generate and inject the name into the synthetized CloudFormation script, matching the specification. Note that <code>functionName</code> is optional and TR-CDK will either TR-ify a provided name or automatically generate a valid one if the user omits it, making sure CloudFormation receives a properly formatted name.</p> 
<h3>Access to “Landing Zone” resources</h3> 
<p>TR’s central AWS Landing Zone team is responsible of inflating a set of standard resources (e.g. VPC, subnets, security groups, Route 53 zones, golden AMIs, etc.) into AWS accounts that are made available to application development teams.</p> 
<p>Through module augmentation (shown earlier), the TR-ifier defines the function <code>trContext()</code> which provides access to a context-aware utility. When calling this function on a resource that resides within a <code>Stack</code>, it will return an object that implements <code>StackContext</code> interface.</p> 
<pre><code class="lang-ts">export interface StackContext extends StageContext {
  /** Get access to the TR IVpc */
  readonly vpc: IVpc;

  /** Provides access to standard security groups that are available in all TR accounts */
  readonly securityGroups: trparams.ISecurityGroupsResolver;

  /** Provides access to private and public hosted zones (with numeric digits) that are available in all TR accounts */
  readonly route53: trparams.IRoute53Resolver;

  /** Provides access to TR golden AMIs that are available in all TR accounts */
  readonly goldenAMI: TRGoldenAMI;
}</code></pre> 
<p>The <code>readonly</code> attributes are accessors for the AWS Landing Zones resources listed above. With calls like the following examples, you have a simple way to obtain access to the standard VPC, subnets selections, route 53 private hosted zone, …</p> 
<pre><code class="lang-ts">// Get the IVpc:
const trVpc: IVpc = stack.trContext().vpc;

// Get the private subnets as array
const privateSubnets: ISubnet[] = trVpc.privateSubnets;

// Get the private subnets as SubnetSelection
const privateSubSel: SubnetSelection = trVpc.selectSubnets({
    subnetType: SubnetType.PRIVATE_WITH_EGRESS,
});

// Get the private Route53 hosted zone
const privateHZ = stack.trContext().route53.privateHostedZone;</code></pre> 
<p>You might now wonder how TR resolves the resources and obtain objects implementing <code>IVpc</code>, <code>ISubnet</code>, <code>ISecurityGroup</code>, …</p> 
<p>Instead of using hard-coded resource attributes (e.g. Id, ARN, …) or complex lookups, TR uses CloudFormation’s <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-supplied-parameter-types.html">ability to resolve Systems Manager parameters at execution time</a>, as part of the AWS account initial inflation along with the resources, Systems Manager parameters are registered as well. The parameter names are the same across TR’s AWS accounts, the value contains e.g. the id of the matching AWS Landing Zone standard resource, e.g. <code>/landing-zone/vpc/vpc-id, /landing-zone/vpc/subnets/private-1-id</code>, <code>/landing-zone/vpc/subnets/private-2-id</code>, …</p> 
<p>TR then defined custom <code>IVpc</code>, <code>ISubnet</code>, <code>IHostedZone</code>… implementations and for each function they implemented dynamic resolution of resource attributes via Systems Manager parameters. With this approach, TR obtains portable code that runs on AWS accounts initialized via TR inflation process. There are no hard-coded resource identifiers, and there is no need for lookups via AWS SDK during synthesis.</p> 
<p>As a user of the TR AWS CDK library, TR developers interact with an object implementing the <code>IVpc</code> interface and do not have to care about how to obtain e.g. the VPC-id and subnet ids. The same principle applies to Route53 hosted zones, Golden AMI ids, etc.</p> 
<h3>Application initialization</h3> 
<p>As mentioned previously, one key design principle is to minimize the custom code that a user of TR AWS CDK is required to use compared to using vanilla AWS CDK. This approach leverages existing AWS CDK and reduces the learning curve for developers.</p> 
<p>This is how TR developers initialize an App with vanilla CDK, compared to how they initialize it with TR AWS CDK.</p> 
<pre><code class="lang-ts">// Initialize a vanilla AWS CDK application
const app = new cdk.App()

// Initialize a TR CDK application
const app = TRCdk.newApp({
  segregationId: '123456',
  resourceOwner: 'team@example.com',
  namingProps: { prefix: 'myapp' },
  deploymentEnv: TRDeploymentEnv.DEV
});</code></pre> 
<p>From this point on, the developers can continue using vanilla AWS CDK code, the value returned by TRCdk.newApp(…) is an instance of an extension of CDK’s App class and is fully compatible with it. It, however, injects the TR-ification aspect, manages the tagging process, and initializes contextual information.</p> 
<p>Here and there, e.g. when they need to pass the VPC into a construct, they will need to call TR AWS CDK code via the <code>trContext()</code> entry point that is exposed on CDK constructs through TypeScript’s module augmentation feature, but that’s it! 99% of the code is vanilla AWS CDK code.</p> 
<p>The <code>segregationId</code>, <code>namingProps</code>, and <code>deploymentEnv</code> attributes are used for multiple purposes like formatting resource names and tagging resources.</p> 
<h3>Standardized Tagging</h3> 
<p>TR defines tagging standards, there are mandatory tags (e.g. for attribution to a specific product asset and for tracking resource ownership), and there are optional tags (e.g. for specifying resources that belong to different services within the same product asset).</p> 
<p>The <code>segregationId</code>, the <code>resourceOwner</code>, and <code>deploymentEnv</code> attributes are used to set mandatory tags using CDK’s built-in functionality for tagging.<br /> TR also defines a standardized set of optional tags that can be passed into the application context or set ad-hoc on individual constructs.</p> 
<pre><code class="lang-ts">// Initialize a vanilla AWS CDK Application
const app = new cdk.App()

// Initialize a TR CDK application
const app = TRCdk.newApp({
  segregationId: '123456',
  resourceOwner: 'team@example.com',
  namingProps: { prefix: 'myapp' },
  deploymentEnv: TRDeploymentEnv.DEV
  optionalTRTags: {
    financialId: '123456789',
    projectName: 'my-project',
    serviceName: 'ServiceX',
    environmentName: 'Dev environment for ServiceX'
  }</code></pre> 
<p>This approach maintains consistency in the use of tag names and setting the values, it happens automatically behind the scenes and will be applied to the taggable constructs. No copy-pasting of tag definitions like in AWS CloudFormation, no issues dealing with CloudFormation’s inconsistent syntax for tag declarations, no forgetting of tagging resources.</p> 
<h2>Conclusion</h2> 
<p>In this post, we discussed how the monorepo approach to AWS CDK development, centered around the core module, has significantly improved the infrastructure management at Thomson Reuters. By providing well-architected L3 constructs, standardizing and simplifying AWS resource creation, they’ve reduced errors, enhanced governance, and accelerated development.</p> 
<p>The core module’s ability to enforce policies, standardize naming and tagging, and provide access to TR-managed resources makes it an invaluable tool for teams working with AWS infrastructure at Thomson Reuters.</p> 
<p>To get started with AWS CDK and build your CDK solutions, check out the <a href="https://docs.aws.amazon.com/cdk/v2/guide/home.html">AWS CDK Developer Guide</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/Danilo.jpg" width="120" />
  </div> 
  <p style="text-align: left;"><b>Danilo Tommasina</b> is a Distinguished Engineer at Thomson Reuters. With over 25 years of experience working in technology roles ranging from Software Engineer, over Director of Engineering and now as Distinguished Engineer. As a passionate generalist, proficient in multiple programming languages, cloud technologies, DevOps practices and with engineering knowledge in the ML space, he contributed to the scaling of TR Labs’ engineering organization. He is also a big fan of automation including but not limited to MLOps, LLMOps processes and Infrastructure as Code principles.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/Lalit.png" width="120" />
  </div> 
  <p style="text-align: left;"><b>Lalit Kumar B</b> is an Associate Cloud &amp; AI Solutions Architect at Thomson Reuters with over 15 years of experience in various technology roles, including Software Engineer, Database Engineer, DevOps Architect, and Solutions Architect, and now as an AI Architect in Platform Engineering. He helped scaling AWS CDK within TR through the ‘tr-cdk-lib’ solution which is an enterprise-grade centralized library of patterns. He enjoys tackling complex challenges and prioritizing effectiveness over efficiency.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/profile_vshhuynh.jpg" width="120" />
  </div> 
  <p style="text-align: left;"><b>Vu San Ha Huynh</b> is a Solutions Architect at AWS with a PhD in Computer Science. He helps large Enterprise customers drive innovation across different domains with a focus on AI/ML and Generative AI solutions.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/10/16/wrightpk.jpg" width="120" />
  </div> 
  <p style="text-align: left;"><b>Paul Wright</b> is a Senior Technical Account Manger, with over 20 years experience in the IT industry and over 7 years of dedicated cloud focus. Paul has helped some of the largest enterprise customers grow their business and improve their operational excellence. In his spare time Paul is a huge football and NFL fan.</p> 
 </div> 
</footer>
