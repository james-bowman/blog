+++
title = "Continuous delivery tool landscape"
categories = ["DevOps","Development"]
tags = ["cd","devops","development","tools"]
draft = true
description = "An overview of some of the tools supporting continuous delivery and devops practices"
date = "2017-01-30T08:34:52Z"
image = "/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg"
#image = "/post/cdlandscape/P1060052_edited-1.jpg"
+++

I have been having a lot of discussions recently about DevOps and Continuous Delivery.  There seems to be a lot of confusion regarding what these terms really mean.  Many people seem to think DevOps is about automation or, more specifically, automated deployments.  Continous Delivery seems to often get confused with Continuous Integration with the terms often being used interchangeably.

There are of course many definitions available on the internet but I find it useful to think of DevOps as a culture of collaboration and continuous improvement facilitating optimisation of the value stream.  Continuous Delivery is an approach allowing developed software to be deployed at any time aiming for faster and more frequent deployments. Both are mutually compatible and usually go hand in hand.

Unfortunately, trying to have a conversation about fuzzy and nebulous things like culture and principles whilst people are looking at tools tends to get greeted with blank expressions.  However, without considering the broader context, the different tools available and the roles they perform, it is likely an inappropriate tool may be selected. With this in mind, I thought it might be useful to visualise the broader CD/DevOps tool landscape and ecosystem to provide some, more concrete, context around the tool(s) being immediately explored and the specific problems each one seeks to address.  

You can see the complete visualisation here: 

<a href="/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg">
	<img src="/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg" alt="Continuous delivery tool landscape Jan 2017">
</a>

I divided the landscape up into 5 high-level phases broadly aligned to a generic application lifecycle: Collaborate, Build, Test, Deploy and Run.  Within each stage, I then attempted to categorise the types of tools available.  Some tools within the same category address slightly different problems and can be considered complimentary to one another e.g. Terraform and Puppet/Chef.  Conversely, some of the tools e.g. MS Team Foundation Server, Go CD, Docker, etc. could appear in multiple categories.  Where this is the case, I have tried to place them once, in the primary category for which they are known.  

## What is not covered

Whilst it would be great to show the entire landscape some compromises had to be made and so some categories of tools were omitted.  These include:

- Service Discovery and global Configuration stores e.g. Consul, ZooKeeper, etcd, etc.
- Security management and monitoring tools (Privileged Account Management, intrusion detection, secret management and certificate management) e.g. CyberArk, Snort, Tripwire, Fortify, Vault, Let’s Encrypt, etc.
- Static code analysis tools i.e. cyclometric complexity, coverage, quality, standards, etc.
- Programming languages, tools and frameworks e.g. compilers, IDEs, Frameworks like DropWizard/Nancy/etc.
- Mocking tools for testing e.g. Mockito
- Quality Management tools e.g. HPE QC
- Release Management tools e.g. LaunchDarkly
- Cloud vendor specific tools & toolchains e.g. Cloudformation and CodeDeploy for AWS
- Platform/Device specific development toolchains e.g. Native mobile apps, IoT, etc.

Have you used other tools that belong in this visualisation?  Does the visualisation resonate with your own experiences and have you used some of the tools shown?  Are there any categories you feel missing?  If so, please comment and share your thoughts and experiences. 