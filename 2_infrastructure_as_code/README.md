# Infrastructure as code

## Getting started with Cloudformation

### What is cloud DevOps

We will deploy the following services
![](README.assets/2022-05-23-08-36-43.png)

Material is available here: https://github.com/udacity/nd9991-c2-Infrastructure-as-Code-v1

DevOps aims to solve the following problems:

- unpredictable deployments
- mismatched environments
- configuration drift

DevOps offers you a set of tools and best practices to solve these problems, for example:

- Configuration management tools such as Chef, Puppet, Ansible are designed to deploy configuration changes to servers
- CI/CD tools such as Jenkins or Circle CI allow to bring code safer and faster to production

### What is CloudFormation

- is a declarative instead of an imperative language
  - e.g. when you define a network and a server, AWS will automatically deploy the network first
  - you can also define dependencies manually, but do this only if required

### CloudFormation vs Terraform

Sources:

- https://www.porscheinformatik.com/en/cloudformation-vs-terraform/
- https://www.cncf.io/blog/2021/04/06/cloudformation-vs-terraform-which-is-better/

|                                     | Terraform                           | Cloudformation                      |
| ----------------------------------- | ----------------------------------- | ----------------------------------- |
| can identify current infrastructure | yes                                 | only if deployed via cloudformation |
| open source                         | yes                                 | no                                  |
| cloud-agnostic                      | yes                                 | no, only AWS                        |
| modularity                          | modules, even many third-party ones | nested stacks                       |
| supports rollbacks                  | no                                  | yes                                 |

### Tips for upcoming labs

- use U.S. west2 region (Oregon)!

### Setting up AWS CLI

```bash
# check version
aws --version
# check configuration (if not: aws configure)
aws configure list
# check access by listing s3 buckets (no error!)
aws s3 ls
# check access by listing IAM users
aws iam list-users
```

### Creating a VPC

- simply google cloudformation VPC for the syntax
- create a YAML file
- a stack in AWS Cloudformation is just a group of resources

```yaml
Description: Some dummy description
Resources:
  MyPersonalVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: String
      # the followings ones are optional
      EnableDnsHostnames: Boolean
      EnableDnsSupport: Boolean
      InstanceTenancy: String
      Ipv4IpamPoolId: String
      Ipv4NetmaskLength: Integer
      Tags:
        - Tag
```

```bash
# to create stack
aws cloudformation create-stack --stack-name ourfirsttest --region us-west-2 --template-body file://VPC_stack.yam

# to update stack
aws cloudformation update-stack ...
```

### excursion: Setup EC2 account with aws-cli

- EC2 accounts already come with the aws-cli tool installed
- setup an EC2 server
- allow SSH access only from your machine
- attach Administrator Policy to EC2 machine
- -> You have another way of using the CLI (not sure of the advantage though?!)

### Bonus: Create an EC2 instances

- rather simple
- recommended VS Code extensions:
  - [CloudFormation](https://marketplace.visualstudio.com/items?itemName=aws-scripting-guy.cform) for scaffolding
  - [CloudFormation Linter](https://marketplace.visualstudio.com/items?itemName=kddejong.vscode-cfn-lint) for highlighting problems

## Infrastructure Diagrams

Simply draw the following diagram in lucidchart (and explain again the components)

### NAT Gateway

- = Network Address Translation (NAT) Gateway
- provides _outbound_ internet access to resources in private subnets
- "translates" inbound public traffic to private traffic?
- Remember: needs to be public itself, therefore placed in a **public** subnet!

### Autoscaling

- needs more than one subnet
- can be used for..
  - high availability (distribute traffic across _healthy_ servers across AZs)
  - high elasticity (scale instances)
- in other words: **autoscaling group is not necessary**!

### Security group

- manages traffic at the resources or _server_ level
- same security group can be _assigned_ to multiple resources

### Route tables

- is about how to _move_ traffic and NOT about allowing or denying traffic!
- For allowing and denying in-/outbound traffic use...
  - access control list (ACL) at the (sub)network level traffic
  - security groups at the resource group
- is associated to e.g. subnets
  - private subnet is defined by a route table rule that all traffic goes to virtual private gateway
- guides routing [FROM resource](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html), not TO resource. However, [return traffic follows the same path](https://docs.aws.amazon.com/prescriptive-guidance/latest/inline-traffic-inspection-third-party-appliances/outbound-traffic-inspection-nat-gateway.html). Also mentioned at a later time "it relays the response back".
  - Route tables can be associated with (internet or virtual private) gateways, likely redirecting inbound traffic
  - Route tables can be associated with subnets, likely redirecting outbound traffinc
- redirects packages addressed to destination to the target
  - in case of multiple matching rules, the rule with the [most specifc destination pattern will be chosen](https://stackoverflow.com/a/51875099/2135504)

**Example**

![](README.assets/2022-06-03-08-02-50.png)

**in this case**

- allow private subnets to reach the internet by redirecting traffic to outside the VPC to the NAT in the public subnet (and from thereon to the internet gateway)
- for public subnets in/outbound traffic outside the VPC is already allowed.

### Reminder: Why Internet GateWay (IGW)?

- because VPC is shut off from internet by default.
- You [cannot assign elastic IPs without an IGW](https://serverfault.com/a/929042/527676).

### Reminder: How to make resources private? [[SO Post]](https://stackoverflow.com/a/72526324/2135504)

- Problem: We want to deny global inbound traffic, but still allow outbound traffic.
- Solution proposals
  - Network Access Control List (NACLs) affect inbound and outbound traffic the same way, therefore don't work for this scenario. Also rarely recommended anyhow
  - Security groups would [solve this problem](https://stackoverflow.com/a/61181364/2135504), but you'd need to set them up for each resource
  - Two separate subnets, one with a connection to IGW; the other with a connection to a NAT gateway

### Bonus: Architecture diagrams

The [AWS Reference Architecture Page](https://aws.amazon.com/architecture/?cards-all.sort-by=item.additionalFields.sortDate&cards-all.sort-order=desc&awsf.content-type=*all&awsf.methodology=*all&awsf.tech-category=*all&awsf.industries=*all) showcases several proven diagrams, which can be copied, among them e.g. the [WordPress architecture ](https://github.com/aws-samples/aws-refarch-wordpress).

### Final picture

![](README.assets/2022-06-07-08-45-34.png)

## Networking Infrastructure

Goal: Recreate the following architecture diagram

![](networking_infrastructure/AWSWebApp.jpeg)

### Workflow

- create initial stack (start with only VPC resource)
- add one resource at a time and update stack
  - potentially use bash helper functions below
  - you could also use "describe stack" to check whether stack already exist and use either create or update

```bash
# create stack
aws cloudformation create-stack \
--stack-name $1 \
--template-body file://$2  \
--parameters file://$3 \
--region=us-east-1

# update stack
aws cloudformation update-stack --stack-name $1 --template-body file://$2  --parameters file://$3 --region=us-east-1

# create with capabilities
aws cloudformation create-stack --stack-name $1 --template-body file://$2  --parameters file://$3 --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" --region=us-east-1
```

### Lessons learned

- renaming *can* create issues
  - Problem: Could not delete Internet Gateway Attachment after renaming because NAT gateways already had public IP addresses assigned
  - Solution: Keep names the same or delete full stack
- NAT gateway is slow to create

## Servers and Security Groups

Based upon the previous lesson, we now create...
- security groups (firewalls) for load-balancer and web-server
- autoscaling group
- EC2 launch configuration
- load balancer

### Lessons learned

- Reminder:
  - Autoscaling group requires a Launch config (LC) OR a Launch template (LT)
    - LT are newer than LC and are recommended now to..
      - get all latests features
      - get versioning
    - LC has the benefit that it is immutable
    - LT can be generated FROM LC

## Storage and Databases

# Deploy a high-availability web app using CloudFormation
