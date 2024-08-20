---
layout: post
title:  "[GSoC] Final report for Google Summer of Code"
date:   2024-08-20
---

# Google Summer of Code - Final report

After many weeks of studying and preparing a self-hosted CI, it is now time to summarize everything that
has been done, pack it in a final report for Google Summer of Code, and review what was done right or wrong
and project the next steps to be followed.

## The proposal

My application for kworkflow was about self-hosted CI with Jenkins - It was initially desired that kworkflow had
its own CI lab for deploying very sophisticated tests that could even involve hardware (thus, requiring
a self-hosted environment for more control of testing nodes). The motivation for testing hardware components
for testing waned over time, but the need for independence with its own CI infrastructure persisted among
kworkflow maintainers. Therefore, I've focused a lot more on other test suites than kworkflow's integration
test - the original main motivation for the self-hosted CI. Ensuring the basics worked well and a replacement
to GitHub Actions would be possible in production seemed much more important as development continued.

The idea of hosting it at the University of São Paulo is still a thing, but worrying about properly 
implementing it there stopped once I started using Jenkins Configuration as Code (will be explained
later on), since I can test everything locally and deploy when it is necessary.

A last point was made in the proposal about preparing the same infrastructure to host the Jenkins server
to also handle telemetry data. Unfortunately, my mentors told me that the telemetry details were still
being discussed and that it was better to focus only on the CI for this GSoC.

## Commentary about the project development overall

This project was somewhat different from most GSoC applications. I wouldn't be contributing
directly with source code from the kworkflow repository, but implementing something else. A self-hosted
server implementation for CI wouldn't follow the conventional Pull Request workflow, right?

This was a very difficult first step to overcome for my application. How could I prove something was being worked on, if my contributions wouldn't be registered on the source code? As time passed, some 
solutions appeared to minimize this problem.

The first idea was to invest time in blog posts. It is a very useful way to prove you've studied a certain
concept, as you explain it yourself, and also report what was done for the sake of the project. I've written
a range of posts, ranging from explaining Jenkins and what I've learned about it along the way, design
ideas on how the CI server would be implemented, and step-by-step descriptions of how things were done
for the kworkflow CI. When this report reaches the timeline section, redirections to other blog posts
will be presented for those seeking a more technical overview of each milestone.

The other solution appeared naturally: to register code on secondary repositories. Since Jenkins, in a
multi-branch Pipeline context, gets the Pipeline instructions from the source code, a lot of experimenting
and Pipeline implementation was registered at my fork of kworkflow. Also, a very promising way to track
progress showed itself as I dived into the configuration of the Jenkins server: I noticed early on 
that the configuration could be written as text. This would not only be very useful for the cited needs
but a strong way to make the server more reproducible. Having all of Jenkins configurations registered
as code meant a very easy deployment of a Jenkins server with the desired traits. 

A GitHub repository, containing the `docker-compose.yml` that deploys the containerized Jenkins server,
and the configuration as code that would be applied to it was made public and worked on during the last
months. It is expected that it will soon be integrated into the kworkflow organization on GitHub.

My kworkflow fork, where I designed the Jenkinsfile Pipelines (on branches `master`, `unstable`, and `reviewdog`)
can be found [here](https://github.com/MarceloSpessoto/kworkflow). The configuration as code of the
Jenkins server can be found [here](https://github.com/MarceloSpessoto/jenkins-kw-infra).

With these measures, it was much easier to track my progress. It was also fundamental to keep constant
communication with my mentors. I tried to show my progress at an almost weekly frequency to my mentor
David in voice calls. It helped me answer questions regarding the next steps and the maintainer's intentions
with the CI project itself.

There were also weekly voice calls involving kworkflow maintainers and contributors. These were not focused
on GSoC, but there were moments where I could show my progress and get feedback from everyone. I participated
in most of these meetings.

## Timeline

Let's recap what has been done in this GSoC (Click each topic to read its blog post with greater detail).

+ [My first step was to experiment with Jenkins and implement a basic code coverage pipeline](first-gsoc)
+ [Then, I found out about Jenkins plugins for configuring the server as Code](implementing-jenkins-as-code)
+ [I spent a good time studying about different configurations of Jenkins agents](jenkins-agents)
+ [I decided to implement a Docker Cloud to provide agents](configuring-the-docker-cloud-for-kw)
+ [I polished the overall server so the containerized pipelines could be put in production](minor-polishments)

## Results

In the end, a Jenkins server, completely configured as code was implemented. The only workflow from
GitHub Actions that were not completely migrated were `integration test`. A prototypal Vagrantfile
deploying a VM as a static agent is present in the Jenkins server repository but will be developed
further as the `integration test` gets into a more advanced state.

The remaining tests didn't involve privileged actions such as invoking new containers, and, therefore,
could be very easily ported in container environments from a Docker Cloud. These are in a final state
before production. There are minor problems, however:

+ The `signal_manager.sh` test fails on Jenkins and passes on GitHub Actions and most local
development environments. It is very curious that it only fails if Jenkins triggers the job, but, if
manually running the `signal_manager.sh` test on the same testing container environment, it passes.
After talking with David, he told me some "bad" unit tests may work on specific and
undefined environments. We'll still have to discuss if this test falls into this category and should
be worked on, or if it can be fixed.
+ The shellcheck pipeline detects a category of linting problem that is not detected on GitHub Actions (SC2319).
More investigation will be conducted to understand what is really happening.

## Next steps

Of course, the work for this Google Summer of Code is far from over and will be worked on after the
final submission:

+ I still have to investigate the problems with the unit tests and shellcheck.
+ The Jenkins configuration repository will be put under the kworkflow organization on GitHub.
+ The Jenkins server will be put on production.
+ Some minor improvements could be made for the Jenkins server, such as storing build artifacts and
statistics.
+ The `ìntegration test` pipeline will be worked on alongside the development of the own test suite.

## Final thoughts

This opportunity to tackle Google Summer of Code was fantastic! It enabled me to learn many different
practices regarding DevOps and Continuous Integration. It also forced me to work directly with tools
like Docker (I've spent a huge time working with Dockerfiles and Docker Compose environments), and Jenkins.

My mentors were great. Not only have I received lots of feedback from them, but they also helped me getting
into GSoC, after all: David read many iterations of my proposal and suggested many improvements, which was
important to get me approved for the program, in the first place.

My journey with kworkflow is not close to an end. I will not only keep an eye on the remaining tasks from
GSoC, but will also contribute more with the source code and upcoming projects related to kworkflow.

