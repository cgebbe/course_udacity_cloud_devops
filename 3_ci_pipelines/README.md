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

### Sharing information

To share information between jobs, there are three ways:

- Cachevariables!
- Workspace
- Secret keeper

#### Cache

- are deleted after pipeline finishes
- great for build artifacts used in later stages
- is a key-storage. To ensure key uniqueness use CI variables
- cache is immutable - once created cannot be changed

```yaml
# in job1
- save_cache:
    key: v1-my-project-{{ checksum "project.clj" }}
    paths:
      - ~/.m2

# in job2
- restore_cache:
    keys:
      - v1-my-project-{{ checksum "project.clj" }}
      - v1-my-project-
```

#### Workspace

- lasts around 15 days
- only folder and filename, no key-storage.
  - Allows to use globs!
- is mutable

```yaml
# job1
- persist_to_workspace:
    root: /tmp/workspace
    paths:
      - target/application.jar
      - build/*

# job2
- attach_workspace:
    at: /tmp/workspace
```

```yaml
# full example
version: 2.1

jobs:
  save_hello_world_output:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - run: echo "hello world" > ~/output.txt
      - persist_to_workspace:
          root: ~/
          paths:
            - output.txt

  print_output_file:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - attach_workspace:
          at: ~/
      - run: cat ~/output.txt

workflows:
  my_workflow:
    jobs:
      - save_hello_world_output
      - print_output_file:
          requires:
            - save_hello_world_output
```

#### Secret keeper

- to save information not as files, but more direct
- options
  - [HashiCorp's Vault](https://www.vaultproject.io/)
    - provides Web UI + CLI tool
  - [MEMSTASH](https://www.memstash.io/)
    - = lightweight service for short-term key storage
    - provides PUT and GET access via curl and Web UI tool
  - [KVdb REST API](https://kvdb.io/)
    - KVdb API lets you store multiple keys-values in a bucket

### Reusable Code

To reuse CI/CD code, there are several options:

- YAML Anchors and Aliases
  - provided via YAML format (i.e. nothing CircleCI specific)
  - helpful for reusing commands
- CircleCI `commands` section

### job failures

To react on job failures, use the `when: on_fail` job parameter (e.g. for rollbacks!).

```yaml
steps:
  - run:
      name: Testing application
      command: make test
      shell: /bin/bash
      working_directory: ~/my-app
      no_output_timeout: 30m
      environment:
        FOO: bar

  - run: echo 127.0.0.1 devhost | sudo tee -a /etc/hosts

  - run: |
      sudo -u root createuser -h localhost --superuser ubuntu &&
      sudo createdb -h localhost test_db

  - run:
      name: Upload Failed Tests
      command: curl --data fail_tests.log http://example.com/error_logs
      when: on_fail
```

### Create a CI pipeline

Example pipeline using three stages:

- compile
- test
- anaylze (security audit)

```yaml
version: 2.1

jobs:
  build:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - checkout
      - run: npm i
      - run: npm run lint
  test:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - checkout
      - run: npm i
      - run: npm run test
  analyze:
    docker:
      - image: circleci/node:13.8.0
    steps:
      - checkout
      - run: npm audit

workflows:
  my_workflow:
    jobs:
      - build
      - test:
          requires:
            - build
      - analyze:
          requires:
            - test
```

## Enabling Continuous Delivery with Deployment Pipelines

### Useful metrics

- **Lead Time to Production**
  - = (Time at Successful Prod Deployment) - (Time at Completion of Feature Grooming)
  - Shows how CI/CD is impacting overall delivery time, from the point the team hears about the feature to the point at which it is done (deployed to production). Easy metric to collect if using task management system to track feature grooming and deployments.
- **Rollback Rate**
  - = (Total Rollbacks) / (Total Deployments)
  - Shows the quality of our deployments. Of course, rate should be low because previous stages should filter out defected builds. This metric is a leading indicator for the confidence of the business in the dev team's ability to delivery.
- **Time to Failure**
  - = (Time at Failure Discovery) - (Time at Build Start)
  - Shows how quickly we find failures. The lower the better.
- **Production Uptime**
  - = (Total Production Working Time) / (Total Time)
  - Shows the amount of time we are taking production down because of botched deployments or due to our chosen deployment strategy.
- **Failed Pipeline Cost**
  - Various calculations including job run time and resources created
  - Shows the estimated amount of money spent on a failed build. Encourages us to put cheaper jobs earlier in the pipeline.

### Configuration Management Tools

Tools like these are great for everything that happens _AFTER_ the server instances are running.

#### Options

- [Chef](chef.io)
  - Depends on agent to be installed. Very mature.
- [Puppet](ouppet.com)
  - easier to work with than chef
  - Requires master "puppet master" server. Performance issues.
- [Salt](saltstack.com)
  - Keeps inventory on a central server.
- [Ansible](ansible.com)
  - Most popular. Very fast. Agentless.

Ansible vs. Terraform:

- Ansible is better suited at [adding software or updates in an already configured environment while Terraform works best for maintaining a steady state without much intervention](https://www.techrepublic.com/article/terraform-vs-ansible/#:~:text=Both%20use%20an%20agentless%20system,modules%20are%20removed%20after%20execution.)
- [Terraform is an Infrastructure as Code platform, whereas Ansibile is a configuration management tool](https://spacelift.io/blog/ansible-vs-terraform)
  - Configuration management is a process of keeping the applications and dependencies up to date
  - orchestration as Day 0 activity and configuration management as Day 1 activity
- [It is recommended](https://spacelift.io/blog/ansible-vs-terraform) to follow the immutable infrastructure approach, where _Terraform takes care of the infrastructure management, and Ansible helps apply the changed configuration_. This is also known as the Blue/Green deployment strategy, where the risk of configuration failure is reduced.

### Design an Ansible Playbook

Ansible scripts are called playbooks. Each playbook contains

- name = Name of the file.
- hosts = Identifies the target machines / hosts.
- tasks = Contains the ordered series of commands to run on the identified hosts. Sometimes, it contains Modules, which are like library functions.
- roles = These are self-contained "child" Playbooks that are used to bring modularity in complex orchestration.

#### Roles

- are the actual meat of the script.
- each role is a directory in `./roles/`
- each role contains subdirectories for different tasks
  - **tasks**
    - Main list of tasks that the role executes
    - contains "modules" such as shell, apt, npm, file, git, script, copy, unarchive, systemd
  - **files** = Files that the role deploys
  - handlers = Handlers, which may be used within or outside this role
  - library = Modules, which may be used within this role
  - defaults = Default variables for the role
  - vars = Other variables for the role
  - templates = Templates that the role deploys
  - meta = Metadata for the role, including role dependencies
- each subdirectory must contain a `main.yml`

```bash
├── main1.yml      # Playbook file (Ignore file name)
└── roles
       ├── configure-prometheus-node-exporter
       │   └── tasks
       │       └── main.yml
       └── configure-server
              └── tasks
                  └── main.yml
```

```yaml
# main1.yml
---
- name: "configuration play."
  hosts: web
  user: ubuntu
  gather_facts: false
  vars:
    - ansible_python_interpreter: /usr/bin/python3
    - ansible_host_key_checking: false
    - ansible_stdout_callback: yaml

  pre_tasks:
    - name: "wait 600 seconds for target connection to become reachable/usable."
      wait_for_connection:

    - name: "install python for Ansible."
      become: true
      raw: test -e /usr/bin/python3 || (apt -y update && apt install -y python3)
      changed_when: false

    - setup:
  roles:
    - configure-prometheus-node-exporter
    - configure-server1
```

```yaml
# roles/configure-server/tasks/main.yml
---
- name: "upgrade packages."
  become: true
  apt:
    upgrade: "yes"

- name: "install dependencies."
  become: true
  apt:
    name: ["nodejs", "npm"]
    state: latest
    update_cache: yes

- name: "install pm2"
  become: true
  npm:
    name: pm2
    global: yes
    production: yes
    state: present
```

### Build an Ansible Inventory File

- lists all target servers
- simple file which can be created via e.g. AWS cli or terraform, see below

```bash
echo "[all]" > inventory
aws ec2 describe-instances \
   --query 'Reservations[*].Instances[*].PublicIpAddress' \
   --output text >> inventory
```

### Control a Remote Machine with an Ansible Playbook

Shows to write an ansible script replicating the [Apache installation tutorial](https://riptutorial.com/apache/example/5607/-ubuntu--simple-hello-world-example)

```bash
#setup files, then  run
ansible-playbook root.yml -i inventory --private-key ~/Desktop/udacity.pem
```

Didn't complete the exercise to write an ansible playbook for the [nodejs installation](https://www.howtoforge.com/tutorial/nodejs-ubuntu-getting-started/) as even locally setup was rather lenghty (included compiling nodejs from source code?!). Yeah, [the solution](https://github.com/udacity/nd9991-c3-hello-world-exercise-solution/blob/main/roles/setup/tasks/main.yml) simply uses `apt install npm nodejs` and `npm install pm2`, where [pm2](https://www.npmjs.com/package/pm2) is kind of a server with an included load balancer?!

### CD jobs: Overview

![](README.assets/2022-07-24-22-03-30.png)

- Create Infrastructure
- Configure Infrastructure
- Deploy Production Artifacts
- Smoke Testing
- Rollback
- Promoting to Production

### CD job: Create Infrastructure

when we want to create new infrastructure for our code to deploy to and then route traffic to (sequentally). This job needs:

- step: checkout code, running cloudformation script
- environment variables: AWS credentials
- image: AWS CLI
- filter: only run job on _master_ pipelines

See [my solution](https://github.com/cgebbe/circleci-demo-javascript-express/tree/5cad4acc4d826c2a8541507fb43b7e147d93c33b/.circleci).

### CD job: Configure Infrastructure

Infrastructure configuration

### CD job: Deploy Production Artifacts

### CD job: Smoke Testing

### CD job: Rollback

### CD job: Promoting to Production

```bash
S3_BUCKET_NAME="my-bucket-811466289151"
aws cloudformation deploy \
--template-file cloudfront.yml \
--stack-name production-distro \
--parameter-overrides PipelineID="${S3_BUCKET_NAME}" \
--tags project=udapeople
```

## Monitoring Environments

### Potential benefits

- Find out when something goes down or degrades (network logs).
- Wake up the right people in an event (app logs).
- Find out an hour before an outage (CPU, memory, disk).
- Use data to predict the next outage in 2 weeks (ML, data science).

### Tipps

- > Liberal collection, conservative reporting
  - this prevents alert fatigue but allows retrospective reports and root cause analysis
- Reactive monitoring vs Proactive monitoring
  - reactive = react upon errors
  - proactive = react upon warning signs (metrics)
    - to e.g. forecast infrastructure costs, seasonal spikes, track bugs

### Three legs of monitoring

- Data with timestamps
- Data Aggregator
  - Options: Graphite, Loggly, Datahog, Prometheus, Logstash, Cloudwatch
- Data visualizer
  - Options: Depends on aggregator: Grafana for Graphite and Prometheus, Kibana for Logstash, datadog, ...

![](README.assets/2022-07-25-09-41-29.png)

### Prometheus

- Prometheus _pulls_ from data sources (rather than receives pushes)
- pull agents are called _exporters_, which are installed or attached to data source.
- Prometheus [supports several exporters](https://prometheus.io/docs/instrumenting/exporters/) for...
  - databases
  - hardware (system metric exporter, special hardwares,. ..)
  - issue tracker and CI
  - messaging systems (kafka, beanstalkd, RabbitMQ)
  - storage
  - HTTP (Apache, Nginx, Blackbox-exporter for website uptime)
  - APIs (AWS ECS, github, gmail, ...)
  - Other logging and monitoring systems
  - ...

Exercise: [Setup hardware metric exporter](https://codewizardly.com/prometheus-on-aws-ec2-part2/) and [EC2 node autodiscovery](https://codewizardly.com/prometheus-on-aws-ec2-part3/).

Exercise: Plot some dummy graphs using internal Prometheus

Exercise: Setup prometheus alertmanager (a small separate program) according to [tutorial](https://codewizardly.com/prometheus-on-aws-ec2-part4/).

## Project

UdaPeople: HR Project

Four sections:

### Explain the fundamentals and benefits of CI/CD to achieve, build, and deploy automation for cloud-based software products.

- 30minutes essay
- To appeal to what makes business people tick, you'll need to focus on benefits that
  - create revenue
  - protect revenue
  - control costs
  - reduce costs
- proposal in document or presentation form
- Your presentation should be no longer than 5 slides
- Your presentation should be in PDF format named "presentation.pdf"

### Utilize Deployment Strategies to design and build CI/CD pipelines that support Continuous Delivery processes.

- You might consider making your repository public so that Circle CI will give you more credits to run builds

- https://github.com/udacity/cdond-c3-projectstarter
- We left a scaffolded/incomplete /.circleci/config.yml file to help you get started with CirlcCI's configuration
- his means, the starter code will not work out of the box on your local machine either.
- It's best if you take screenshots along the way and store them in a folder on your computer until you're ready to turn the project in
- The free account includes 2500 credits per week which equals around **70 builds**. If
- you run out of credits, you can create another account and continue working.
- circle-ci should alert user via e.g. slack in case of failed jobs (oh no, recently also email notification!)

### Utilize a configuration management tool to accomplish deployment to cloud-based servers.

- configure via ansible
- make blue-green deployment at the end

### Surface critical server errors for diagnosis using centralized structured logging.

- add prometheus logging

## Lessons learned in project

### Check connection to database

```bash
# check DNS lookup
nslookup mydb2.cxhga6eksa1z.us-east-1.rds.amazonaws.com

# from https://stackoverflow.com/a/44496546/2135504 (also points to python scripts!)
pg_isready -h mydb2.cxhga6eksa1z.us-east-1.rds.amazonaws.com -d mydb2

# The following command will throw errors for wrong host
# [sql:]vendor://[[user][:password]@][host][:port]/[database][?sqlquery]
sql jdbc:postgresql://postgres@mydb2.cxhga6eksa1z.us-east-1.rds.amazonaws.com --listproc
```

Also pay attention to the security group! The associated security group needs to allow PostgreSQL connections from my CURRENT IP and the circleCI-IP. In contrast, by default it uses the default VPC security group which only allows access within the VPC (despite public access).

```bash
aws cloudformation deploy \
 --template-file cloudfront.yml \
 --stack-name InitialStack\
 --parameter-overrides WorkflowID=udapeople-811466289151
```

### Note: Infrastructure for backend

According to the instructions there should be an error for SCREENSHOT 05

> Fix: You will have to change the AMI ID and KeyPair name in the backend.yml file, as applicable to your AWS account/region, before you push your changes to Github.

However, both the AMI ID and the KeyPair worked fine out of the box.

### AWS dynamo DB as key-value store

```bash
# put item
VAR="my-var"
aws dynamodb put-item --table-name key-value-store --item '{ "key": {"S": "'${VAR}'" }, "value": {"S": "value-for-myvar" }}'

# get item value
VAR="my-key"
RESPONSE=$(aws dynamodb get-item --table-name key-value-store --key '{"key": {"S": "'${VAR}'"}}' --attributes-to-get "value" --output="text")
SUCCESS=${RESPONSE:6}
```

### Analyze content of postgres database

DBeaver was the easiest one to use among the [five most recommended GUI tools](https://scalegrid.io/blog/which-is-the-best-postgresql-gui-2019-comparison/).

### Submission

```yaml
# URL to your public GitHub repository
URL01: https://github.com/cgebbe/cdond-c3-projectstarter

# public URL for your S3 Bucket (aka, your front-end)
URL02: http://udapeople-17904c9.s3-website-us-east-1.amazonaws.com/#/employees

# URL03_SCREENSHOT: cloudfront
URL03: https://d33p89m778wm4m.cloudfront.net

# URL04_SCREENSHOT: backend
URL04: http://34.238.157.34:3030/api/status

# URL05_SCREENSHOT:prometheus
URL05: http://18.234.190.164:9090/targets
```

### Problem with pm2 and environment variables

```bash
export TYPEORM_CONNECTION="postgres"
export TYPEORM_MIGRATIONS_DIR="./src/migrations"
export TYPEORM_ENTITIES='./src/modules/domain/**/*.entity.ts'
export TYPEORM_MIGRATIONS='./src/migrations/*.ts'
export TYPEORM_HOST="mydb2.cxhga6eksa1z.us-east-1.rds.amazonaws.com"
export TYPEORM_PORT="5432"
export TYPEORM_USERNAME="postgres"
export TYPEORM_PASSWORD="mypassword"
export TYPEORM_DATABASE="mydb2"

npm run start

pm2 --update-env start npm -- start

# No repository for "Product" was found. Looks like this entity is not registered in current "default" connection? {"trace":"RepositoryNotFoundError: No repository for \"Product\" was found. Looks like this entity is not registered in current \"default\" connection?\n    at new RepositoryNotFoundError (/home/ubuntu/src/error/RepositoryNotFoundError.ts:10:9)\n
```

### get IP from EC2 instance

```bash
aws cloudformation deploy \
--template-file .circleci/files/backend.yml \
--stack-name "udapeople-backend-123" \
--parameter-overrides ID="123"  \
--tags project=udapeople
```

### Final prometheus configs

All files are stored in `/etc/prometheus`.

**alertmanager.yml**

```yaml
global:
  resolve_timeout: 1m
  slack_api_url: "https://hooks.slack.com/services/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"

route:
  group_by: [Alertname]
  group_wait: 30s
  group_interval: 10s
  repeat_interval: 10s
  receiver: slack-notifications

receivers:
  - name: "slack-notifications"
    slack_configs:
      - channel: "#alerts"
        send_resolved: true
        # text: "<!channel> \nsummary: {{ .Annotations.summary }}\ndescription: {{ .Annotations.description }}"
        text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"
```

**rules.yml**

```yaml
groups:
  - name: Down
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: "critical"
        annotations:
          summary: "Instance {{ $labels.instance }} is down."
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

**prometheus.yml**

```yaml
rule_files:
  - /etc/prometheus/rules.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093

global:
  scrape_interval: 15s
  evaluation_interval: 10s
  external_labels:
    monitor: "prometheus"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    ec2_sd_configs:
      - region: us-east-1
        access_key: AKIA3Z3ZI5776P3ZE7RS
        secret_key: 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
        port: 9100
```
