# Build CI/CD Pipelines, Monitoring & Logging

## Introduction to CI/CD

### Terminology

- Continuous Delivery
  - paradigm / mindset to release frequently and easily
- Continuous Integration
  - merge quickly to a shared master
  - this is about source code
- Continuous Deployment
  - deploy frequently via automated deployments
  - this is about build code

### Principles of Continuous Delivery

From the book ["Continuous Delivery" by Jez Humble and David Farley"](https://www.amazon.com/dp/0321601912?tag=contindelive-20).

- Repeatable Reliable Process
- Automate Everything
- Version Control Everything
- Bring the Pain Forward
- Build-in Quality
- "Done" Means Released
- Everyone is Responsible
- Continuous Improvement

### Tools to use

Circle CI, which monitors e.g. GitHub repositories and launches builds for a new commit. It features:

- testing builds in Docker containers or virtual machines
- deploying targets to enviroments
- monitoring the build in the form of a dashboard and API
- error notifications via e.g. Slack

## Continuous Integration and Continuous Deployment Theory

### Fundamentals of CI/CD

> Continuous Integration (CI) = The practice of merging all developers' working copies to a shared mainline several times a day.

> Continuous Deployment (CD) = A software engineering approach in which the value is delivered frequently through automated deployments

> CI/CD pipeline is a "value factory".

![](README.assets/2022-07-12-21-43-32.png)

> pipeline > stage > job > step

### Benefits of CI/CD

You can automate (and thereby standardize) nearly all of the following things:
![](README.assets/2022-07-12-21-32-53.png)

Exceptions may be:

- coding stays human
- code review is assisted by static analysis
- client test may not be needed if we can build enough confidence
- [Smoke tests](<https://en.wikipedia.org/wiki/Smoke_testing_(software)>) are a subset of test cases that cover the most important functionality of a component or system

### Continuous Delivery as a Paradigm

### CI/CD Best Practices

**Fail Fast**
Set up your CI/CD pipeline to find and reveal failures as fast as possible (cost of failure increases exponentially the later in the deployment process you catch it). The faster you can bring your code failures to light, the faster you can fix them.

**Measure Quality**
Measure your code quality so that you can see the positive effects of your improvement work (or the negative effects of technical debt).

**Only one Road to Production**
Once CI/CD is deploying to production on your behalf, it must be the only way to deploy. Any other person or process that meddles with production after CI/CD is running will inevitably cause CI/CD to become inconsistent and fail.

**Maximum Automation**
If it can be automated, automate it. This will only improve your process!

**Config in Code**
All configuration code must be in code and versioned alongside your production code. This includes the CI/CD configuration files!

### Deployment Strategies

**Big-Bang**
Replace A with B all at once.

- PRO
  - simple to understand
- CON
  - downtime while replacing A with B
  - encourages teams to batch up changes
  - requires more testing and coordination

**Blue Green**
Two versions of production: Blue or previous version and Green or new version. Traffic can still be _routed_ to blue while testing green. Switching to the new version is done by simply shifting _100% traffic_ from blue to green.

- PRO
  - can test new deployment without disturbing old one
  - no downtime
  - allows fast rollback
- CON
  - pay for two parallel versions (at least for a while)

**Canary**
Aka _Rolling Update_, After deploying the new version, start routing traffic to new version little by little until all traffic is hitting the new production. Both versions coexist for a period of time.

- PRO
  - lowers risks
- CON
  - higher infrastructure costs
  - more difficult to setup

**A/B Testing**
Similar to Canary, but instead of routing traffic to new version to accomplish a full deployment, you are testing your new version with _specific subset of users_ for feedback.

- PRO
  - can be used to evaluate feature effects via feedback (to target specific users)
- CON
  - costly method of user acceptance testing (UAT), there are other methods (like feature flags?)
  - even more difficult than canary

### Blue-Green Deployment

There are several router options:

- Load Balancer
  - Instant switch for frontend or backend
  - ideal router in most cases
- Content delivery network (CDN)
  - Instant switch for front-end web apps.
- Domain name system (DNS)
  - A bit slow because of DNS propagation.
- Custom routing in your application
  - only do this if we required, rather complex

A typical CI/CD pipeline for a blue-green deployment could look like this:
![](README.assets/2022-07-12-23-02-10.png)

### Pipeline Building Blocks and Jobs

- Build
  - Everything that has to do with making code executable in production (e.g. linting, compiling, zipping). The goal is to produce an _artifact_.
- Test
  - All automated tests that verify at the code level.
- Analyze
  - Any static analysis on the code or checking of dependencies, e.g. security audits, mutation testing, static code analysis.
- Deploy
  - Anything to do with creating server instances or copying pre-built application files to an instance.
- Verify
  - Any tests that can be run against a running instance of the application, often against a pre-production instance. If verification fails, we can _revert_ the change.
- Promote
  - Replacing the current production environment with the new version which was just built and deployed.
- Revert
  - Rolling back or undoing changes in case any verification fails after deployment.

Example logic pipeline

![](README.assets/2022-07-12-23-49-35.png)

### CI/CD Tools

- On-Prem servers
  - examples
    - Jenkins
    - Gitlab CI/CD (free community edition)
    - TeamCity by JetBrains
- Cloud-based
  - PRO: requirees less time to maintain and configure
  - CON: usually not free (at least for team)
  - examples
    - Bamboo by Atlassian
    - Travis CI (very simple, free for open source)
    - Gitlab
    - Circle CI (very fast)

## Building a Continuous Integration Pipeline

> Continuous Integration is the practice of automating the integration of code changes from multiple contributors into a single software project ([definition from Atlassian](https://www.atlassian.com/continuous-delivery/continuous-integration)).

### Best Practices

CI/CD pipelines...

- need to become everybodys highest priority when the build is broken
- should be considered as a trusted human team member (e.g. does not inform management upon errors)
- should have exactly the same abilities as any member of the team, no less and no more (e.g. special access)
- should enforce team quality rules concerning code style and linting
- communicates useful information (e.g. if it needs fixing)
- shorten feedback loops
- don't require stacks of documentation
- are automated end-to-end

Additionally, the Single Responsibility Principle (SRP) also applies to CI/CD pipelines! Each stage / job / step should have only one _goal_.

### Example pipeline with Circle CI

See [CircleCi getting started guide](https://circleci.com/docs/getting-started).

```yaml
# Use the latest 2.1 version of CircleCI pipeline process engine. See: https://circleci.com/docs/2.0/configuration-reference
version: 2.1

jobs:
  print_hello:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - run: echo hello
  print_world:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - run: echo world

workflows:
  welcome:
    jobs:
      - print_hello
      - print_world:
          requires:
            - print_hello
```

CircleCI offers default [_pipeline values_](https://circleci.com/docs/2.0/pipeline-variables/#pipeline-values) such as `pipeline.id`, `pipeline.project.git_url` or `pipeline.git.branch`, which can be used in the YAML definition.

### Environment Variables

CircleCI offers default environment variables like `CIRCLE_BRANCH`or `CIRCLE_PR_NUMBER`. One can [specify more environment variables via the UI](https://circleci.com/docs/env-vars#setting-an-environment-variable-in-a-project) and then use them in the UI.

The environment variables can be scoped on three levels:

- job
- project (~= pipeline?)
- organization

### Trigger

The first two triggers are the most common ones.

- **Git Branch Commit**
  - Commit or merge to a branch-like master and push changes to the branch in central repository to start a new build.
- **New Pull/Merge Request**
  - Make changes in a branch or fork and create a pull/merge request to trigger a build.
- **API**
  - Make a POST or GET request to an API endpoint to kick off a new build.
- **Schedule**
  - Run a pipeline at a certain time each day or week based on a schedule.
- **Other Pipelines**
  - Another pipeline might finish a job and then trigger another pipeline.
- **Chat Message**
  - Using a chat tool, post a message containing special text in order to trigger a build.
- **Command-Line Tool**
  - Use a command-line tool to configure and start a new build.

#### Example

- in github
  - setup new repository on github
  - under settings > Webhooks, add a new webhook
    - payload URL = backend server to receive request (google.com)
    - select events (e.g. new push event, new issue event) to trigger the webhook
- Go to CircleCI
  - connect to github
  - select new repository > "start building"
  - it automatically adds a webhook

## Enabling Continuous Delivery with Deployment Pipelines

## Monitoring Environments

## Project

UdaPeople: HR Project

### Tipps
