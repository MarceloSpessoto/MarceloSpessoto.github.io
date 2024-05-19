# Applying the Jenkins as Code paradigm

On my first two GSoC weeks, I've dealt with a basic Code Coverage pipeline and implemented it on a 
bare-metal infrastructure. My next expected steps, according to my project schedule, were to start 
implementing the virtual machine and physical device agents. Turns out, however, that this task may be
delayed a little bit more, so I can focus on polishing the Jenkins pipeline foundation.

As I've immersed myself in the Community Bonding period, the proper port of the complete actual GitHub 
Actions CI to Jenkins has emerged as a more important task. From that, we will have a nicer environment
to plan and develop the new testing workflow. Also, the dummy implementations could be seamlessly packed
into the development period, where I will direct all my efforts on this specific issue.

## Applying IaC

Infrastructure as Code (IaC) is a paradigm where the implementation of infrastructure is described in "code",
such as yaml scripts. This way, one can easily automate the deployment of such infrastructure by using the
code instead of setting things up manually.

Doing that for the kworkflow Jenkins pipeline would bring many benefits. Most of the CI configurations
and plugin settings could be saved on a VCS host server, keeping all the DevOps work preserved and ready
for deployment on any bare-metal network.

### First step: setting a Docker Compose environment

First of all, it will be interesting to replace the actual Jenkins bare-metal install with a containerized
service. It makes the service deployment easier and also enables one to set up a specific configuration for
Jenkins as Code Plugin (we'll come into that really soon) and plugin set.

Also, this enables the use of Docker Compose to orchestrate the automated deployment of the Jenkins servers
alongside its Jenkins agent container. 
