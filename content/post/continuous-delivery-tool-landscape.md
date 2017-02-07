+++
title = "Continuous delivery tool landscape"
categories = ["DevOps","Development"]
tags = ["cd","devops","development","tools"]
draft = true
description = "An overview of some of the tools supporting continuous delivery and devops practices"
date = "2017-01-30T08:34:52Z"
socialimage = "/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg"
#image = "/post/cdlandscape/P1060052_edited-1.jpg"
image = "/post/cdlandscape/5666226557_fd58e28368_b.jpg"
imageURL = "https://www.flickr.com/photos/thelearningcurvedotca/5666226557/in/photolist-9CGTKk-4Btkot-4Btktr-87uhSr-8osTy-9svYGE-8j2FPs-4fVeMf-Fppkv-oW9WhF-nFnDEA-4d52wB-cXs2n-k5ZFc-6Snb1-niC99x-6aRnQN-3rThWk-qs68e8-6dFaYQ-5jsdCA-9tmFyT-3acUTk-7ATuoW-gGoJ4j-4BxBSC-cn8Mu-4BxBDu-4h4FfJ-8VpcTK-5wmsZ-3K2DPi-4FEQ1Q-MDu7-6989qm-4pxcPw-KWqRy-4o8YT-iWheoe-4aa7tW-9QkrJh-3Y98RR-5xF9ff-cWwCRo-dj5KU8-nXdRv-jH6mq5-gxRP1f-w46E-4cdkAS"
imageTitle = "Speed"
imageCreator = "Brian Carson"
imageLicenceURL = "https://creativecommons.org/licenses/by-nc-sa/2.0/"
imageLicenceName = "Creative Commons Attribution-NonCommercial-ShareAlike 2.0 Generic license"
+++

I have been having a lot of discussions recently about DevOps and Continuous Delivery.  There seems to be a lot of confusion regarding what these terms really mean.  Is there more to DevOps than automation, and more specifically, automated deployments?  What is the difference between Continous Delivery, Continuous Integration and even Continuous Deployment?

There are of course many definitions available on the internet but I find it useful to think of DevOps as a culture of collaboration and continuous improvement facilitating optimisation of the value stream.  Continuous Delivery is an approach allowing developed software to be deployed at any time aiming for faster and more frequent deployments. Both are mutually compatible and usually go hand in hand.

Unfortunately, trying to have a conversation about fuzzy and nebulous things like culture and principles whilst people are looking at tools doesn't feel particularly helpful.  However, without considering the broader context, the different tools available and the roles they perform, it is likely an inappropriate tool may be selected. With this in mind, I thought it might be useful to visualise the broader CD/DevOps tool landscape and ecosystem to provide some, more concrete, context around the tool(s) being immediately explored and the specific problems each one seeks to address.  

You can see the complete visualisation here: 

<a href="/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg">
	<img src="/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg" alt="Continuous delivery tool landscape Jan 2017">
</a>

I divided the landscape up into 5 high-level phases broadly aligned to a generic application lifecycle: Collaborate, Build, Test, Deploy and Run.  Within each phase, I then attempted to categorise the types of tools available.  Some tools within the same category address slightly different problems and can be considered complimentary to one another e.g. Terraform and Puppet/Chef.  Conversely, some of the tools e.g. MS Team Foundation Server, Go CD, Docker, etc. could appear in multiple categories.  Where this is the case, I have tried to place them once, in the primary category for which they are known.  

## What is not covered

Whilst it would be great to show the entire landscape on one page some compromises had to be made and so some categories of tools were omitted.  These include:

- Service Discovery and global Configuration stores e.g. Consul, ZooKeeper, etcd, etc.
- Security management and monitoring tools (Privileged Account Management, intrusion detection, secret management and certificate management) e.g. CyberArk, Snort, Tripwire, Fortify, Vault, Let’s Encrypt, etc.
- Static code analysis tools i.e. cyclometric complexity, coverage, quality, standards, etc.
- Programming languages, tools and frameworks e.g. compilers, IDEs, Frameworks like DropWizard/Nancy/etc.
- Mocking tools for testing e.g. Mockito
- Quality Management tools e.g. HPE QC
- Release Management tools e.g. LaunchDarkly
- Cloud vendor specific tools & toolchains e.g. Cloudformation and CodeDeploy for AWS
- Platform/Device specific development toolchains e.g. Native mobile apps, IoT, etc.

## Wrap up

The visualisation is not intended to be exhaustive or represent any form of recommendation or endorsement.  It would be interesting to hear other people's perspectives and experiences.  Have you used other tools that belong in this visualisation?  Does the visualisation resonate with your own experiences and have you used some of the tools shown?  Are there any categories you feel missing?  If so, please comment and share your thoughts and experiences. 