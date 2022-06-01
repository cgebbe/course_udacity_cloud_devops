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

## Infrastructure Diagrams

## Networking Infrastructure

## Servers and Security Groups

## Storage and Databases

# Deploy a high-availability web app using CloudFormation
