+++
title = "Continuous delivery tool landscape"
categories = ["DevOps","Development"]
tags = ["cd","devops","development","tools"]
draft = false
description = "An overview of the tools supporting CD (Continuous Delivery) and devops practices in 2017"
date = "2017-01-30T08:34:52Z"
socialimage = "/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg"
#image = "/post/cdlandscape/P1060052_edited-1.jpg"
image = "/post/cdlandscape/5666226557_94ce72fc4e_o.jpg"
imageURL = "https://www.flickr.com/photos/thelearningcurvedotca/5666226557/in/photolist-9CGTKk-4Btkot-4Btktr-87uhSr-8osTy-9svYGE-8j2FPs-4fVeMf-Fppkv-oW9WhF-nFnDEA-4d52wB-cXs2n-k5ZFc-6Snb1-niC99x-6aRnQN-3rThWk-qs68e8-6dFaYQ-5jsdCA-9tmFyT-3acUTk-7ATuoW-gGoJ4j-4BxBSC-cn8Mu-4BxBDu-4h4FfJ-8VpcTK-5wmsZ-3K2DPi-4FEQ1Q-MDu7-6989qm-4pxcPw-KWqRy-4o8YT-iWheoe-4aa7tW-9QkrJh-3Y98RR-5xF9ff-cWwCRo-dj5KU8-nXdRv-jH6mq5-gxRP1f-w46E-4cdkAS"
imageTitle = "Speed"
imageCreator = "Brian Carson"
imageLicenceURL = "https://creativecommons.org/licenses/by-nc-sa/2.0/"
imageLicenceName = "Creative Commons Attribution-NonCommercial-ShareAlike 2.0 Generic license"
+++

I have been having a lot of discussions recently about tooling to support continuous delivery and DevOps practices.  There is an incredible and ever increasing array of tools available for these practices.  Whilst a number of vendors have developed [one-stop solutions or suites of integrated tools](https://dzone.com/articles/continuous-delivery-anti-patterns), many of the tools in the space tend to be tightly focused on addressing a particular problem.  

Unfortunatley this can be confusing and overwhelming, especially to people starting out, making it difficult to know where to start and which tools to consider.  This can also lead to particular tools being used to solve problems where other types of tools may be better suited.  It is therefore important to consider tools within the context of the broader ecosystem and understand the role each one plays and the specific goal or problem(s) they aim to address.  With this in mind, I thought it might be useful to visualise the broader CD/DevOps tool landscape to provide some context around the available tools and how they each fit within it.  

You can see the complete visualisation here: 

{{< figure src="/post/cdlandscape/ContinuousDeliveryToolLandscape.jpeg" link="/post/cdlandscape/ContinuousDeliveryToolLandscape-fullsize.jpeg" alt="Continuous delivery tool landscape Jan 2017" >}}

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