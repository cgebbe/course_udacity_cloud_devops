
```bash
# create-stack or update-stack
# FILESTEM="network"
FILESTEM="servers"

aws cloudformation \
update-stack \
--stack-name "udacity-iac-${FILESTEM}" \
--template-body "file:///mnt/sda1/projects/git/courses/udacity_cloud_devops/2_infrastructure_as_code/networking_infrastructure/${FILESTEM}.yaml"  \
--parameters "file:///mnt/sda1/projects/git/courses/udacity_cloud_devops/2_infrastructure_as_code/networking_infrastructure/${FILESTEM}-params.json" \
--region=us-east-1

# create with capabilities
aws cloudformation create-stack --stack-name $1 --template-body file://$2  --parameters file://$3 --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" --region=us-east-1
```