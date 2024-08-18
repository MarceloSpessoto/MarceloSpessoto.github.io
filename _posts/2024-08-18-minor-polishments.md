---
layout: post
title:  "[GSoC] Some minor last polishments to the Jenkins server"
date:   2024-08-18
---

# Final preparations for GSoC

With the upcoming final submission period for Google Summer of Code, it is time to ensure the Jenkins server
setup is ready to start its migration process to the kworkflow official repository.

## Considerations about the actual state of the infrastructure

During this GSoC experience, many aspects of the project were changed alongside the kworkflows needs for a
self-hosted CI. 

The desire to integrate hardware testing with kworkflow CI tests waned during the last months and the proposal
of a self-hosted infrastructure focused much more on kw having independence over the testing process for new PRs. Therefore, my CI implementation was decided to stick with a single VM node only, replicating GitHub Actions workflow
for integration tests. Also, the production version of the integration test pipeline will be initiated shortly after
the final submission, alongside the final integration test development phase. For now, we'll keep aiming to
polish the other tests that run in containers, so they can be integrated into the official kworkflow repository
as soon as possible.

In this task, I've encountered some interesting implementation details that should be registered here. Firstly,
I will expose the nitty-gritty of the reviewdog application, its use on kworkflow original GitHub Actions workflow
and how it could be exposed to Jenkins.

Then, I will talk about the GitHub Checks plugin, and how it was used to provide a smoother UX for contributors when
submitting their PRs and the bit of research made to implement it in 'as Code' with the Job DSL plugin.

## Integrating reviewdog with Jenkins

[Reviewdog](https://github.com/reviewdog/reviewdog) is a software that can receive output from a linting tool and
produce 'review comments' on a Pull Request page at a VCS host (GitHub or GitLab). It is a quite good addition
to an open-source software, providing a very concise and friendly way of showing linting errors to contribute.
With reviewdog, every linting error will be represented on the PR host as a comment on the problematic piece
of code explaining the problem!

### Porting the `shellcheck_reviewdog.yml` workflow to Jenkins

There are many GitHub Actions available that automatically execute a linting software and set reviewdog
to receive this output. 

The kworkflow project just uses the `reviewdog/action-shellcheck@v1` action to set up everything.

For Jenkins, a small amount of extra work should be done to apply reviewdog PR reviews on a Pipeline.
One should consider the execution of shellcheck itself piping its output to reviewdog and handle the 
proper configuration of reviewdog with ENV vars, including the GitHub Token.

The first peculiarity I encountered when setting this up was piping the shellcheck output directly 
to reviewdog. With Jenkins Pipelines, bash commands are executed with some indirection. The groovy
syntax of Pipelines allows the execution of such commands with the `sh` command receiving the bash
command as argument (example: `sh 'ls -la'`). The problem is, that it doesn't allow piping commands. If
`sh 'cat file.txt' | grep 'word'` is applied, only the `cat file.txt` will be applied. The workaround 
found to solve that was to wrap all of the piping in a bash command. The execution of shellcheck integrated
with reviewdog, at Jenkins, became: 

```
sh '/bin/bash -c "shellcheck -f checkstyle --external-sources --shell=bash --exclude=SC2016,SC2181,SC2034,SC2154,SC2001,SC1090,SC1091,SC2120 src/*.sh | reviewdog -f checkstyle -reporter=github-check"'
```

It is a very long one-line command but necessary. First, the groovy `sh` command tells Jenkins Pipeline
to execute a bash command. The command itself is a `bash` command that will execute the piping command.
It is meant to wrap the piping in a single `bash` command, allowing its execution in `sh`. The content
of the command to be executed by `bash` is the `shellcheck` piping its output to `reviewdog`. There are many
flags configuring both shellcheck and reviewdog accordingly with the original kworkflow setup.

The kworkflow project uses reviewdog to write comments on the linting errors of a PR on its page.

There is also a next little step for reviewdog, regarding how it will authenticate with GitHub to write 
comments on the Pull Request and properly detect branches. This is accomplished with ENV vars, which can
be set on the Jenkins Pipeline environment option, as follows:

```
  agent { label 'shellcheck-agent' }
  environment {
    CI_PULL_REQUEST = "${CHANGE_ID}"
    CI_REPO_OWNER = "MarceloSpessoto"
    CI_REPO_NAME = "kworkflow"
    CI_COMMIT = "${GIT_COMMIT}"
    REVIEWDOG_INSECURE_SKIP_VERIFY = "true"
    REVIEWDOG_GITHUB_API_TOKEN = credentials('github-token')
  }
  stages { ... }
```

`CI_PULL_REQUEST` will get the PR ID to be reviewed and can be easily obtained with `CHANGE_ID`
ENV var that is automatically set by Jenkins. `CI_REPO_OWNER` and `CI_REPO_NAME` are configurations with
self-explanatory names. `REVIEWDOG_INSECURE_SKIP_VERIFY` was a small configuration that doesn't really
affect our case and will likely be disabled in production. 

And the most important of all, `REVIEWDOG_GITHUB_API_TOKEN` which is used to authenticate with GitHub
and write the review comments. It is a simple GitHub Token (The same one you generate to authenticate to
the upstream repo if the remote was set with HTTP over SSH). Also, notice that it was stored in and passed from a
Jenkins credentials to protect its contents in the public Jenkinsfile.

After setting the Pipeline in the forked `kworkflow` repository from the `reviewdog` branch, I've
requested a PR to the unstable to see if Jenkins would be triggered, evaluated `shellcheck` from
the PR'ed `reviewdog` branch, and write a comment over linting mistakes. And it worked!

![Image](assets/images/polishments/Reviewdog-comment.png)

## Configuring the GitHub Checks plugin

When running multiple Jenkins Pipelines, the Checks notifications from GitHub get very messy if only using
the GitHub Branch Source Plugin. The check from a Pipeline will be a single check called 'Jenkins', and
every Pipeline will overwrite the check.

The GitHub Checks plugin enables the configuration of a GitHub Check for each Pipeline, and also disables
the default overridable check from the GitHub Branch Source Plugin. It is pretty easy, on the job configuration,
go to the Branch Source section (this plugin is integration with Branch Source), click on the "Add" button
at the bottom, and select "Configure GitHub Checks". The rest is pretty intuitive.

The last obstacle was to configure it "as Code", using Job DSL. After searching the 
[Job DSL API](https://jenkinsci.github.io/job-dsl-plugin/), I've noticed the yellow square section
suggesting to access the API from `https://<jenkins-server>/plugin/job-dsl/api-viewer/index.html`.
And it helped a lot, since it contains much more documentation, by exhibiting not only basic Job DSL 
options but also the installed plugins on the given Jenkins server. After that, I've found out how
to configure the checks.

## Final thoughts

The Jenkins CI is in a very good state and ready to become production soon. This is an accomplishment that
will define the end of my Google Summer of Code contributions. But of course, there will be much more
after that. I will help with the transition to production and contribute to the integration test CI.
