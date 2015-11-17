+++
categories = ["DevOps", "Development"]
date = "2015-11-11T20:01:19Z"
description = "Run your own private, self-hosted continuous delivery pipeline to build Docker containers using GOGS and Drone"
draft = true
tags = ["cd", "devops", "development", "docker", "containers", "git", "microservices"]
title = "Run your own private CD pipeline with GOGS, Drone and Docker"

+++

Nowdays there are lots of services available on the internet supporting continuous delivery.  Tools such as [GitHub](http://github.com/) and Atlassian's [BitBucket](http://bitbucket.org/) for version control, and automated build, test and deployment pipeline tools like [Travis](http://travis-ci.com), [CircleCI](http://circleci.com), [Codeship](http://www.codeship.com), [Wercker](http://wercker.com) and [Drone](http://drone.io).  Whilst they offer service plans to suit most people in terms of price and privacy, there are reasons an organisation might be reluctant to use public services for these purposes.  In this post, I explore how to create a privately hosted continuous delivery pipeline using open source tools.  

In the last couple of years there seems to have been an explosion in the number of CI/CD services for automated build and deployment pipelines e.g. [Travis](http://travis-ci.com), [CircleCI](http://circleci.com), [Codeship](http://www.codeship.com), [Wercker](http://wercker.com) and [Drone](http://drone.io).  This new generation of CI/CD tools differ from more traditional tools like [Jenkins](https://jenkins-ci.org/) and [TeamCity](https://www.jetbrains.com/teamcity/) in that they push configuration for the build out of the tool and back into the source code repository where it is versioned along with the code it is building.  Typically the CI tool configuration takes the form of a YAML file in the root of the repository with tool specific configuration settings.  Furthermore, many of these newer tools use containerisation technologies like Docker to create isolated, repeatable and disposable build environments for different technology stacks.  This enables the service to support a wide range of technology stacks simultaneously without requiring the superset of all build dependencies to be installed on each build agent.  This can be particularly valuable with microservice based architectures where services are often developed in differing languages and technology stacks. 


pipeline as code
programming in your CI/CD tool - hold
https://www.thoughtworks.com/radar/techniques/programming-in-your-ci-cd-tool


## Installing docker

## Setup GOGS

Setup gogs/gogs
mapping /data volume to a host location

	docker run -d --restart=always --name=gogs -p 10022:22 -p 80:3000 \
		-v /var/gogs:/data gogs/gogs:latest

Launch web application and enter domain and IP address and port of container running on host

Create an application authentication token

Create an authorisation

Create a repo



## Setup Drone

	docker run --volume <local location for SQLLite DB>:/var/lib/drone \
		--volume /var/run/docker.sock:/var/run/docker.sock \
		--restart=always --publish 8000:8000 --detach \
		--env REMOTE_DRIVER=gogs --env REMOTE_CONFIG="http://<gogs URL>:<gogs HTTP port>" \
		--env DRONE_GOGS_URL="http://<gogs URL>:<gogs SSH/Git port>" \
		--env DRONE_GOGS_SECRET=<application token from gogs> \
		--name=drone drone/drone:latest

Login to drone console and select and activate build repo.  This will setup the appropriate web hook in the GOGs project so that all subsequent commits will trigger a build.

Looks like you need to restart Drone container to refresh list of repositories from GOGS!

Don’t forget proxy settings - HTTP_PROXY, etc.

## Setup registry

	docker run -d -p 5000:5000 --restart=always --name registry \
		-v ~/Docker/registry:/var/lib/registry registry



``` Dockerfile
FROM nginx
COPY public /usr/share/nginx/html
```

Run Docker daemon in insecure mode (only necessary if not running on localhost (Docker toolbox/boot2docker/docker-machine) and not using certs.
s
make changes to daemon:

inside boot2docker

	docker-machine ssh default
	sudo -i

open or create the file /var/lib/boot2docker/profile and add the following line:	EXTRA_ARGS="--insecure-registry myinternaldocker"	

After the change you need to restart the docker daemon:	

leave boot2docker and restart the entire virtual machine:	

	boot2docker down
	boot2docker up

Drone Wall

	docker run -d -p 3000:3000 -e API_SCHEME=HTTP -e API_DOMAIN=192.168.99.100 -e API_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0ZXh0IjoiamFtZXMuZWR3YXJkLmJvd21hbiIsInR5cGUiOiJ1c2VyIn0.DajYihXxoFN2gR7vtqbwb71xfXo6Y4gaOMIFt9Zh1I8 -e API_PORT=80 scottwferg/drone-wall

