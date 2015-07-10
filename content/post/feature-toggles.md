+++
date = "2015-07-03T13:00:42+01:00"
draft = true
title = "feature toggles"
tags = [ "cd", "devops", "branching", "dev" ]
categories = [
  "Development",
  "DevOps",
]

+++

I always dread the phrase "branching strategy".  To me, the very fact we need a strategy suggests there are too many branches.  

Pushing the complexities of parallel development down into a tool rather than making it a first class concern within the code itself where it is explicit and clearly visible to everyone.

How many branches is too many?  From a CD perspective: more than one!

<!--more-->

# When to deploy?
Stories in progress but not complete/finished

## Possible options:
### Code freeze at end of iteration
- forms a natural sync point to deploy and release at end of iteration
- idle time
- waiting for everything to be complete - cycle time = lead time
- How do you release fixes to bugs in production?
- Support branches
- Allows a secondary route to push changes (e.g. bug fixes) to production independently of and more quickly than standard release route
- can be trunk based development for new features and fixes are applied to support branch more closely resembling current state of prod.
- Merge overhead of merging fixes back into trunk (fixes need to be applied in two places - support branch and trunk).
- Assumes a (relatively) long time between deployments
- Lot of overhead if deploying once a day
- Some fixes may take longer than a day to implement in which case it makes more sense just to deploy from trunk
- Extra complexity/special cases in build/deploy pipeline to facilitate automated build, test and deploy from a branch.

### Feature branches
- Allows more frequent releases from trunk using a standard release path
- Selectively merge new features into trunk when they are ready for release
- Complexity pushed down into VCS and deferred until later in the delivery cycle.
- Merge overhead
- Drift between various different branches
- When do you deploy and when do you merge branches into trunk?

### CD
- trunk based development
- Makes parallel development activities explicit in the code itself.
- Makes deployment and release first class concerns that must be considered upfront by developers.
- Deployment and release of features is independent
- Allows frequent deployments from trunk.

## What are toggles?

We are really talking about a collection of strategies for keeping changes out of the execution path until we are ready to release them, so that we can keep deploying the code base with other changes in the meantime.  We can selectively choose when each feature is released independently of deployment.

The frequency of deployment should not be limited to the rate at which we release new features.

Includes some combination of:
toggles
branch by abstraction
unreachable code
API/contract versioning
Fine grained, independently deployable and rev’able components.
Loose coupling and Postel’s law
(temporary!) parallel running and duplication of code/data.

## Types of toggles
Ops
Dynamically toggle-able
Story
Toggle placed in code at the start of every story so new code is not in execution path.
Story can be deployed independently of release
Once complete, story can be toggled on and rebuilt, tested and deployed.
Once happy in prod, toggle can be removed.
All deployments (even bug fixes aka “patch releases”) go through same repeatable process
Canary releases
Feature

## How to automatically test stories with toggles
locally, unit testing
functionally at service level
selenium tests
exploratory testing

testing what is about to be deployed rather than what has been developed
only relevant when there is a gap between deployed and dev’ed (inventory)

