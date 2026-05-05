---
title: "Standardizing construct properties with AWS CDK Property Injection"
url: "https://aws.amazon.com/blogs/devops/standardizing-construct-properties-with-aws-cdk-property-injection/"
date: "Thu, 05 Mar 2026 17:39:48 +0000"
author: "Marco Frattallone"
feed_url: "https://aws.amazon.com/blogs/devops/category/management-tools/aws-cloudformation/feed/"
---
<p>Standardizing CDK construct properties across a large organization requires repetitive manual effort that scales poorly as teams and repositories grow. Development teams working with AWS Cloud Development Kit (<a href="https://docs.aws.amazon.com/cdk/">AWS CDK</a>) must apply the same configuration properties across similar resources to meet security, compliance, and operational standards but manual configuration leads to drift, maintenance burden, and compliance gaps. In this post, you learn how to use Property Injection, a feature introduced in AWS CDK v2.196.0, to automatically apply default properties to constructs without modifying existing code.</p> 
<h2>The Challenge of Infrastructure Standardization</h2> 
<p>Organizations implementing infrastructure as code face a fundamental tension between developer productivity and operational consistency. CDK provides abstractions for defining cloud resources, but ensuring compliance with organizational security policies, compliance requirements, and operational standards requires repetitive manual configuration.<br /> Consider this scenario: an organization’s security policy requires that all SecurityGroups disable outbound traffic by default. Development teams must apply these settings to every <code>SecurityGroup</code>:</p> 
<pre><code class="lang-typescript">
new SecurityGroup(stack, 'api-sg', {
  vpc: myVpc,
  allowAllOutbound: false,        // Required by security policy
  allowAllIpv6Outbound: false     // Required by security policy
});

new SecurityGroup(stack, 'db-sg', {
  vpc: myVpc,
  allowAllOutbound: false,        // Same configuration repeated
  allowAllIpv6Outbound: false     // Same configuration repeated
});
</code></pre> 
<p>This manual approach creates four specific problems:</p> 
<ul> 
 <li><strong>Configuration drift:</strong> Teams omit required properties or apply them inconsistently</li> 
 <li><strong>Maintenance burden:</strong> Policy updates require coordinated changes across multiple repositories and teams</li> 
 <li><strong>Developer friction</strong>: Repetitive configuration tasks slow development velocity and increase cognitive load</li> 
 <li><strong>Compliance gaps:</strong> Manual processes introduce human error, creating security or compliance violations</li> 
</ul> 
<p>Custom construct libraries address these challenges but require refactoring every construct instantiation in existing code and create learning curves for development teams already familiar with standard CDK patterns.</p> 
<h2>Introducing Property Injection</h2> 
<p><strong>AWS CDK Property Injection addresses these challenges by automatically applying default properties to constructs without requiring changes to existing code.</strong></p> 
<p><a href="https://docs.aws.amazon.com/cdk/v2/guide/blueprints.html">Property Injection</a> is a feature introduced in AWS CDK v2.196.0 that intercepts construct creation and automatically applies organizational defaults. With this approach, you can enforce standards consistently while preserving existing development workflows and code patterns.</p> 
<p>After implementing Property Injection, the same <code>SecurityGroup</code> creation requires only the vpc parameter, security defaults are applied automatically:</p> 
<pre><code class="lang-typescript">// Your existing code remains unchanged
new SecurityGroup(stack, 'my-sg', {
  vpc: myVpc
  // Security defaults applied automatically by Property Injection
});</code></pre> 
<p>The key benefits of this approach include:</p> 
<ul> 
 <li><strong>Zero-impact adoption:</strong> Existing CDK code continues to work without modification</li> 
 <li><strong>Centralized policy management:</strong> Standards are defined once and applied automatically</li> 
 <li><strong>Consistent enforcement:</strong> Policies are applied uniformly across all applications and teams</li> 
 <li><strong>Reduced maintenance overhead:</strong> Policy updates require changes in only one location</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_25079" style="width: 886px;">
 <img alt="This diagram shows the five-step Property Injection process in a clear two-column format. The left column outlines each process step, while the right column shows the corresponding implementation details with properly formatted TypeScript code. The flow demonstrates how CDK intercepts SecurityGroup creation, applies organizational security defaults through property injectors, merges them with developer-specified properties, and creates a fully configured SecurityGroup that meets both developer requirements and organizational standards." class="wp-image-25079 size-full" height="1035" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/01-cdk-property-injection-mechanism.png" width="876" />
 <p class="wp-caption-text" id="caption-attachment-25079">Figure 1: CDK Property Injection Mechanism</p>
</div> 
<p>Property Injection operates transparently within CDK, intercepting construct creation to apply predefined defaults before merging them with any properties explicitly provided by developers. This ensures that organizational standards are consistently applied while maintaining the flexibility for developers to override defaults when specific use cases require it.</p> 
<h2>Understanding the Implementation Approach</h2> 
<p>Property Injection works by implementing the <code>IPropertyInjector</code> interface, which allows you to define default properties for specific construct types. These injectors are registered with CDK stacks and automatically apply their defaults during construct instantiation.<br /> The implementation follows three steps: define the defaults you want to apply, register the injector with your stack, and let CDK handle the automatic application of these defaults to matching constructs.</p> 
<h3>Implementation Guide</h3> 
<p>This section shows you how to implement Property Injection for <code>SecurityGroup</code> constructs.</p> 
<h4>Step 1: Create a Property Injector</h4> 
<p>Create a class that implements the IPropertyInjector interface:</p> 
<pre><code class="lang-typescript">import { IPropertyInjector, InjectionContext } from 'aws-cdk-lib';
import { SecurityGroup, SecurityGroupProps } from 'aws-cdk-lib/aws-ec2';

export class SecurityGroupDefaults implements IPropertyInjector {
  readonly constructUniqueId: string;

  constructor() {
    this.constructUniqueId = SecurityGroup.PROPERTY_INJECTION_ID;
  }

  inject(originalProps: SecurityGroupProps, context: InjectionContext): SecurityGroupProps {
    return {
      // Apply organizational defaults
      allowAllIpv6Outbound: false,
      allowAllOutbound: false,
      // Original properties override defaults when specified
      ...originalProps,
    };
  }
}</code></pre> 
<h4>Step 2: Add the Injector to Your Stack</h4> 
<p>Apply the injector to your CDK stack:</p> 
<pre><code class="lang-typescript">import { Stack } from 'aws-cdk-lib';
import { SecurityGroupDefaults } from './security-defaults';

const stack = new Stack(app, 'MyStack', {
  propertyInjectors: [
    new SecurityGroupDefaults()
  ]
});</code></pre> 
<h4>Step 3: Use Constructs Normally</h4> 
<p>Create constructs as usual. The injector applies defaults automatically:</p> 
<pre><code class="lang-typescript">// This SecurityGroup receives the injected defaults:
// - allowAllOutbound: false
// - allowAllIpv6Outbound: false
new SecurityGroup(stack, 'my-sg', {
  vpc: myVpc
});

// You can override defaults when necessary
new SecurityGroup(stack, 'special-sg', {
  vpc: myVpc,
  allowAllOutbound: true  // Overrides the injected default
});</code></pre> 
<div class="wp-caption alignnone" id="attachment_25081" style="width: 1119px;">
 <img alt="This side-by-side comparison shows the difference between manual configuration and Property Injection. The left side (Before) shows three SecurityGroup definitions, each requiring manual specification of allowAllOutbound: false and allowAllIpv6Outbound: false, leading to repetitive code, inconsistency risk, and maintenance burden. The right side (After) shows the same SecurityGroups created with the VPC parameter alone after a one-time Property Injection setup, demonstrating the DRY principle, consistent defaults, and reduced maintenance." class="wp-image-25081 size-full" height="675" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/02-cdk-code-before-after.png" width="1109" />
 <p class="wp-caption-text" id="caption-attachment-25081">Figure 2: CDK Code Before vs After Property Injection</p>
</div> 
<h2>Property Injection vs L2 Constructs</h2> 
<p>You can achieve the same enforcement of default properties by creating custom <a href="https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_lib">L2 constructs</a> with built-in defaults. However, Property Injection is better suited for standardizing existing codebases without refactoring, while L2 Constructs are better suited for new projects where you want custom APIs and multi-resource abstractions.</p> 
<div class="wp-caption alignnone" id="attachment_25083" style="width: 1073px;">
 <img alt="This decision tree guides the selection between Property Injection and L2 Constructs for CDK standardization. Starting with existing CDK applications, it evaluates willingness to accept potential breaking changes from new defaults. If breaking changes are acceptable or no existing code exists, it assesses whether custom APIs, naming improvements, or multi-resource patterns are needed beyond simple defaults. The tree leads to three outcomes: Property Injection (blue) for transparent defaults with existing code compatibility, L2 Constructs (orange) for custom APIs and purpose-built abstractions, or a Hybrid approach (green) combining both techniques for maximum flexibility." class="size-full wp-image-25083" height="683" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/03-decision-tree.png" width="1063" />
 <p class="wp-caption-text" id="caption-attachment-25083">Figure 3: Decision Tree – Property Injection vs L2 Constructs</p>
</div> 
<h2>Implementation Comparison</h2> 
<p>Consider an application with multiple <code>SecurityGroup</code> instantiations that need standardized security defaults.</p> 
<p><strong>L2 Construct approach</strong> requires creating a custom construct and updating each instantiation:</p> 
<pre><code class="lang-ts">// Step 1: Create custom L2 construct
export class SecureSecurityGroup extends SecurityGroup {
  constructor(scope: Construct, id: string, props: SecurityGroupProps) {
    super(scope, id, {
      allowAllOutbound: false,
      allowAllIpv6Outbound: false,
      ...props
    });
  }
}

// Step 2: Update each instantiation throughout your codebase
// Change from:
new SecurityGroup(stack, 'sg1', { vpc: myVpc })
new SecurityGroup(stack, 'sg2', { vpc: myVpc })
new SecurityGroup(stack, 'sg3', { vpc: myVpc })

// To:
new SecureSecurityGroup(stack, 'sg1', { vpc: myVpc })
new SecureSecurityGroup(stack, 'sg2', { vpc: myVpc })
new SecureSecurityGroup(stack, 'sg3', { vpc: myVpc })</code></pre> 
<p><strong>Property Injection approach</strong> requires one-time stack configuration:</p> 
<pre><code class="lang-ts">// Step 1: Add injector to stack configuration
stack.propertyInjectors = [new SecurityGroupDefaults()];

// Step 2: Existing SecurityGroup calls receive defaults automatically
new SecurityGroup(stack, 'sg1', { vpc: myVpc })  // Gets defaults
new SecurityGroup(stack, 'sg2', { vpc: myVpc })  // Gets defaults  
new SecurityGroup(stack, 'sg3', { vpc: myVpc })  // Gets defaults</code></pre> 
<h2>Key Differences</h2> 
<p>Property Injection works with existing construct calls, requiring no changes to how developers instantiate SecurityGroups or other constructs. This approach overrides constructs from external libraries and can be implemented without modifying existing code. Developers continue using familiar CDK APIs without learning new interfaces.</p> 
<p>L2 Constructs require updating all constructor calls throughout your codebase. This approach cannot modify third-party construct creation since you must change each instantiation to use your custom construct. Implementation requires refactoring existing code and developers must learn your custom construct APIs instead of standard CDK interfaces. L2 constructs serve multiple purposes beyond complex business logic – simple L2 constructs provide domain-specific naming conventions and cleaner APIs, while complex L2 constructs orchestrate three or more resources and implement business rules.</p> 
<h2>When to Choose Each Approach</h2> 
<p><strong>Choose Property Injection when you need to standardize existing infrastructure.</strong> Property Injection excels in scenarios where you already have CDK applications deployed and need to apply consistent defaults retroactively. Property Injection works transparently with existing code, requiring no changes to how developers instantiate constructs. This makes it useful when you have existing CDK applications that you want to standardize without disrupting current development workflows.</p> 
<p>Property Injection also solves the challenge of applying defaults to constructs from third-party libraries. Since you cannot modify external library code, Property Injection enforces organizational standards on any construct type, regardless of its source. Additionally, when you want to implement standards without changing existing code, Property Injection operates at the framework level, automatically applying defaults during construct instantiation without requiring developers to modify their existing implementations.</p> 
<p><strong>Choose L2 Constructs when you need custom APIs or multi-resource patterns.</strong> L2 Constructs provide the right abstraction when you want to create purpose-built interfaces that differ from standard CDK APIs. This includes simple wrappers with domain-specific naming, complex business logic, validation rules, or multi-resource orchestration patterns. L2 Constructs excel when you want to create opinionated APIs that simplify common patterns by hiding complexity behind intuitive interfaces.</p> 
<p>L2 Constructs suit new application development where you can design the API from the start. This approach creates purpose-built abstractions that match your organization’s specific use cases and terminology. Unlike Property Injection, which applies defaults to existing construct APIs, with L2 Constructs you can design entirely new APIs that directly represent your business domain and operational patterns.</p> 
<h2>Implementation Patterns</h2> 
<h3>Stack Integration Methods</h3> 
<p>The CDK provides two methods for adding Property Injectors to stacks:</p> 
<div class="wp-caption alignnone" id="attachment_25088" style="width: 1107px;">
 <img alt="This diagram demonstrates two methods for adding Property Injectors to CDK stacks. Method 1 (blue) shows adding injectors directly in the Stack constructor’s propertyInjectors array. Method 2 (orange) shows using PropertyInjectors.of(stack).add() after stack creation. Both methods produce identical results with green checkmarks indicating success. The diagram includes usage examples showing normal SecurityGroup instantiation (blue) that inherits defaults automatically, and override scenarios (orange) where developers explicitly override injected defaults. The bottom section shows the resulting CloudFormation output: default SecurityGroups have empty egress rules (green), while overridden ones include outbound traffic rules (orange)." class="wp-image-25088 size-full" height="784" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/04-cdk-stack-integration.png" width="1097" />
 <p class="wp-caption-text" id="caption-attachment-25088">Figure 4: CDK Stack Integration Methods</p>
</div> 
<p><strong>Method 1: Stack Constructor</strong></p> 
<pre><code class="lang-ts">const stack = new Stack(app, 'MyStack', {
  propertyInjectors: [new SecurityGroupDefaults()]
});</code></pre> 
<p><strong>Method 2: PropertyInjectors.of()</strong></p> 
<pre><code class="lang-ts">const stack = new Stack(app, 'MyStack');
PropertyInjectors.of(stack).add(new SecurityGroupDefaults());</code></pre> 
<p>Both methods produce the same result. Choose the method that best fits your existing code structure. For more details, see the <code>PropertyInjectors</code> API documentation.</p> 
<h3>Organization-Wide Implementation</h3> 
<p>For organization-wide standardization, create a shared library of injectors:</p> 
<pre><code class="lang-ts">// @myorg/cdk-injectors package
export const ORGANIZATION_INJECTORS: IPropertyInjector[] = [
  new SecurityGroupDefaults(),
  new LambdaFunctionDefaults(),
  new S3BucketDefaults(),
];

// Teams import and use the shared injectors
import { ORGANIZATION_INJECTORS } from '@myorg/cdk-injectors';

const stack = new Stack(app, 'TeamStack', {
  propertyInjectors: ORGANIZATION_INJECTORS
});</code></pre> 
<h3>Scope Hierarchy</h3> 
<p>Property Injectors can be applied at different levels in the <a href="https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_tree">CDK construct tree</a>:</p> 
<div class="mceTemp"> 
 <p><img alt="This diagram illustrates the three-level hierarchy of Property Injector scopes in CDK with integrated resolution examples. The App level (blue) shows a BucketInjector ‘b1’ that applies globally, with an example showing how Stack2 buckets use this injector. The Stage level (green) demonstrates a FunctionInjector ‘f1’ that applies to all stacks within the stage, including an example of Stack1 functions using this injector. The Stack level (orange) shows two stacks: Stack1 with its own BucketInjector ‘b2’ that overrides the app-level injector, and Stack2 with no injectors that inherits from parent scopes. The Resolution Rules box (green) explains that CDK searches from most specific (stack) to most general (app), with the first match winning per construct type. Arrows show the hierarchical relationship between scopes." class="wp-image-25089 size-full" height="754" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/05-cdk-scope-hierarchy.png" width="1124" /></p> 
 <ul class="wp-caption alignnone" id="attachment_25089" style="width: 1124px;">
  Figure 5: CDK Scope Hierarchy &amp; Injector Resolution
 </ul> 
</div> 
<ul> 
 <li><strong>App level:</strong> Applies to all stacks in the application</li> 
 <li><strong>Stage level:</strong> Applies to all stacks within a specific stage</li> 
 <li><strong>Stack level:</strong> Applies only to constructs within a specific stack</li> 
</ul> 
<p>CDK searches for applicable injectors starting from the construct’s immediate parent scope and moving upward. The first matching injector for each construct type is used.</p> 
<h2>Best Practices</h2> 
<p>When implementing Property Injection, begin with high-impact constructs like SecurityGroups, VPCs, and Lambda functions that require repetitive configuration.</p> 
<p>These constructs have the highest frequency of misconfiguration and the most direct compliance impact, making them the most valuable targets for early adoption.</p> 
<p>Document your defaults by explaining what properties your injectors provide and why. Include examples and link to relevant policies that drive the requirements. With this documentation, developers can understand standards and make informed override decisions.</p> 
<p>Write automated tests using CDK testing utilities to verify that injectors apply expected defaults. Test both standard scenarios and cases where developers override properties to prevent regressions when updating injector logic.</p> 
<p>Version injectors carefully using semantic versioning principles because changes affect all applications. Coordinate updates across teams and provide migration guides for breaking changes or changes to default values.</p> 
<p>Design override mechanisms so that developers can handle edge cases while benefiting from organizational standards. Property Injection operates as defaults, not restrictions, so design injectors to merge gracefully with developer-specified properties.</p> 
<h2>Limitations and Considerations</h2> 
<p>Property Injection operates as a default mechanism rather than a compliance enforcement system. Developers retain the ability to override injected properties, which means organizations cannot rely solely on Property Injection for strict compliance requirements. For teams that need mandatory compliance, combine Property Injection with <a href="https://docs.aws.amazon.com/cdk/v2/guide/aspects.html">CDK Aspects</a> or <a href="https://docs.aws.amazon.com/config/">AWS Config rules</a> to validate and enforce standards.</p> 
<p>The feature works exclusively with L2 constructs, as documented in the official AWS CDK guidance. The <code>IPropertyInjector</code> interface targets specific L2 construct types, and <a href="https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_l1_using">L1 (CloudFormation) constructs</a> use different instantiation patterns that bypass the property injection mechanism entirely. Organizations with L1 construct usage need alternative standardization approaches.</p> 
<p>Property Injection introduces debugging complexity because injected properties do not appear directly in application code. Developers troubleshooting construct behavior must understand which injectors apply to specific construct types and how those injectors modify properties. This hidden behavior requires documentation that lists each injector, the properties it sets, and the policy it enforces, along with clear naming conventions to maintain code clarity.</p> 
<p>The feature requires CDK v2.196.0 or later, which affects adoption timelines for organizations using older CDK versions. Teams must plan upgrade paths and test compatibility before implementing Property Injection across their applications.</p> 
<h2>Conclusion</h2> 
<p>Property Injection provides a mechanism for applying consistent default properties to CDK constructs without requiring changes to existing code. This approach reduces repetitive configuration, improves consistency, and simplifies maintenance of CDK applications.<br /> Property Injection is the right choice for organizations that need to standardize construct configurations across existing codebases while preserving developer workflows. When combined with proper testing and documentation, Property Injection becomes a reliable foundation for infrastructure governance across your organization.</p> 
<h3>About the authors:</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="//d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/04/20231126_103410-150x150.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Put Cheung</h3> 
  <p style="text-align: left;">Put Cheung is a Senior Software Development Engineer at AWS Security. He is a part of a team that is making it easier for builders to configure AWS Resources securely. AWS CDK Property Injection is an important step toward this goal.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="120" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2026/03/03/rico.png" width="120" />
  </div> 
  <h3 class="lb-h4">Rico Huijbers</h3> 
  <p style="text-align: left;">Rico Huijbers is a Software Engineer at Amazon Web Services. He is extremely lazy and is therefore on a quest to eradicate the need for repetitive manual work from software engineering. Rico loves working on AWS CDK—it’s the tool he wishes he had 5 years earlier.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2025/01/10/CW_8565.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Marco Frattallone</h3> 
  <p style="text-align: left;">Marco Frattallone is a Senior Technical Account Manager at AWS focused on supporting Partners. He works closely with Partners to help them build, deploy, and optimize their solutions on AWS, providing guidance and leveraging best practices. Marco focuses on helping Partners adopt emerging AWS services and translate technical capabilities into business outcomes. Outside work, he enjoys outdoor cycling, sailing, and exploring new cultures.</p> 
 </div> 
</footer>
