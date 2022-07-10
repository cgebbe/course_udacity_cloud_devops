# Deploy a Web App using Cloudformation

The Udagram project is currently accessible on the [this URL](http://udaci-webap-2rtpnnan14yf-133479692.us-east-1.elb.amazonaws.com/) and should read:

> it works! Udagram, Udacity served from S3

## Architecture diagram

![](architecture_diagram.png)

## Creation steps

### Create network infrastructure

Create the network infrastructure in exactly the same way as in the lecture:

```bash
ROOT_PATH="file:///mnt/sda1/projects/git/courses/udacity_cloud_devops/2_infrastructure_as_code/project"

# setup network infrastructure
FILESTEM="network"
aws cloudformation \
create-stack \
--stack-name "udacity-iac-${FILESTEM}" \
--template-body "${ROOT_PATH}/${FILESTEM}.yaml"  \
--parameters "${ROOT_PATH}/${FILESTEM}-params.json" \
--region=us-east-1
```

### Create S3-bucket with content

Create the S3 bucket via the CLI instead of a CloudFormation script as advised in the lecture.

```bash
# mb for make bucket
aws s3 mb s3://my-930812052113-bucket --region us-east-1

# create file
echo 'it works! Udagram, Udacity served from S3!' > index.html
aws s3 cp index.html s3://my-930812052113-bucket/index.html
```

### Create servers

Setup your [EC2 instance with automatic aws cli access](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-metadata.html).

```bash
# setup server infrastructure
FILESTEM="servers"
aws cloudformation \
create-stack \
--stack-name="udacity-iac-${FILESTEM}" \
--template-body="${ROOT_PATH}/${FILESTEM}.yaml"  \
--parameters="${ROOT_PATH}/${FILESTEM}-params.json" \
--region=us-east-1 \
--capabilities CAPABILITY_NAMED_IAM
```
