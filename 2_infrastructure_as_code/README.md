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

- renaming _can_ create issues
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
- Debugg `unhealthy`

### Relation between Load Balancer and Autoscaling Group

- A load balancer is a device that simply forwards traffic, evenly across a group of servers, known as a Target Group.
- The problem is, we can’t specifically name those servers, because if they are part of an Auto Scaling group, this means that they can come and go as demand for your application increases or decreases.
- The way around this is, using the TargetGroupARNs (ARN=Amazon Resource Name) property of the Auto Scaling group. In this way, all servers launched via the autoscaling policy will be registered in the target group from the load balancer.

![](README.assets/2022-06-12-22-10-08.png)

![](README.assets/2022-07-04-09-36-50.png)

### Jump box to access servers in private subnet

- Create a server called "jumpbox" in a public subnet
  - with a security group that only allows SSH inbound traffic from YOUR IP.
- SSH into your public jumpbox server
  - `ssh user@ip -i path/to/key.pem`
- Setup SSH for your private servers
- download SSH key to your computer
- secure copy the SSH key for your private servers to your jumbbox via `scp`
  - `scp -i path/to/key.pem path/to/local/src user@ip:/path/to/dst

### Default username of ec2 instances

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html

Get the default user name for the AMI that you used to - launch your instance:

- For Amazon Linux 2 or the Amazon Linux AMI, the user name is ec2-user.
- For a CentOS AMI, the user name is centos or ec2-user.
- For a Debian AMI, the user name is admin.
- For a Fedora AMI, the user name is fedora or ec2-user.
- For a RHEL AMI, the user name is ec2-user or root.
- For a SUSE AMI, the user name is ec2-user or root.
- For an Ubuntu AMI, the user name is ubuntu.
- For an Oracle AMI, the user name is ec2-user.
- For a Bitnami AMI, the user name is bitnam
- Otherwise, check with the AMI provider.

### TODO

- access EC2 instance via SSH -> works with
  - Opening port 80 in security group (this is where apache server runs)
  - `MapPublicIpOnLaunch: true` for all subnets
  - `ssh -i ~/Desktop/VocareumKey2.pem ubuntu@18.234.149.37`
  - `service apache2 status`
- access to one EC2 instance (currently not possible)
- access via load balancer
- fix unhealthy status

## Storage and Databases

TODO

- 4 Create aurora database in UI
- 8 create bucket and upload file via console
  - ah, no, simply `aws s3 cp /src/path s3://dst/path`

### RDS Database

It is recommended to setup the database using the console (or UI) to avoid accidental deletion when running the cloudformation script. Also, there's little benefit in using cloudformation as it is a one-time thing.

If you choose to use a cloudformation script, be sure to use _Retention Policy_ to prevent data deletion. This property keeps a service even if the entire stack of infrastructure is marked for removal. In CloudFormation, the syntax is DeletionPolicy: retain.

### S3

- again, rather create using console or UI
- Your servers don't need credentials to access S3 provided they have an IAM role assigned.
- We recommend you choose RDS as opposed to installing a database in your own servers that you have to manage and back up yourself.

## Deploy a high-availability web app using CloudFormation

### Supporting material

- https://github.com/udacity/nd9991-c2-Infrastructure-as-Code-v1/tree/master/project_starter
  - although text should say "it works! Udagram, Udacity"
- see also old YAML starter code here: https://knowledge.udacity.com/questions/123109

```bash
#!/bin/bash
apt-get update -y
apt-get install apache2 -y
systemctl start apache2.service
cd /var/www/html

# Create a dummy page...
echo "Udacity Demo Web Server Up and Running!" > index.html

# ...or download s3 content.
apt-get install unzip awscli -y
aws s3 cp s3://udacity-demo-1/udacity.zip .
unzip -o udacity.zip
```

### IP for load balancer

[An Application Load Balancer cannot be assigned an Elastic IP address (static IP address). However, a Network Load Balancer can be assigned one Elastic IP address for each Availability Zone it uses. If you do not wish to use a Network Load Balancer, you can combine the two by putting the Network Load Balancer in front of the Application Load Balancer.](https://stackoverflow.com/a/55243777/2135504)

### x86 vs ARM processor architecture

- [x86 for maximum performance, ARM for power efficiency](https://vasexperts.com/blog/telecom/whats-better-x86-or-arm/)
- ARM [is a simpler architecture leading to lots of power save features](https://stackoverflow.com/a/14795541/2135504)
  - therefore, ARM used in mobile phones mainly
- The [computational performance of AWS’s Arm EC2 instances is similar to that of the x86_64 instances](https://www.infoq.com/articles/arm-vs-x86-cloud-performance/)
- ARM instances are significantly cheaper

### Pick instance type and AMI

> You'll need two vCPUs and at least 4GB of RAM. The Operating System to be used is Ubuntu 18. So, choose an Instance size and Machine Image (AMI) that best fits this spec.

As instance type, pick [t3.medium](https://aws.amazon.com/ec2/instance-types/?trk=1c70ffc0-2c7c-41d2-802c-21145de63ecb&sc_channel=ps&sc_campaign=acquisition&sc_medium=ACQ-P|PS-GO|Brand|Desktop|SU|Compute|EC2|DACH|EN|Text|EU&s_kwcid=AL!4422!3!536451478217!b!!g!!%2Baws%20%2Bec2%20%2Binstance%20%2Btypes&ef_id=Cj0KCQjwn4qWBhCvARIsAFNAMii3atZCSM5GVSgO8L7OxWRl0mY3sEA09hcuXap1kGr5ao3XqCEeMEoaAhb1EALw_wcB:G:s&s_kwcid=AL!4422!3!536451478217!b!!g!!%2Baws%20%2Bec2%20%2Binstance%20%2Btypes) with 2vCPUs and 4GB Ram. Ah, submission criteria says [t3.small suffices](https://review.udacity.com/#!/rubrics/2556/view).

For AMI see AMI catalog

- ami-0f1aec6bc79ad857c
  - Deep Learning AMI (Ubuntu 18.04) Version 62.0
  - likely overkill, choose another
- ami-0729e439b6769d6ab (64-bit (x86)) / ami-0ca951dd3a6f8cbfc (64-bit (Arm))
  - Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
  - even free tier eligible

### Submission criteria

- Use LaunchConfiguration (instead of template)
- Provide a working server with URL
- SSH key: There shouldn’t be a ‘keyname’ property in the launch config
