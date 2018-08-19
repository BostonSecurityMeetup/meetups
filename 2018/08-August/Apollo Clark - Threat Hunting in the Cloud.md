# Approach
1. Inventory Management
2. Access Management
3. Configuration Management
4. Patch Management
5. Logging and Monitoring
6. Alerts
7. Automated Remediation

- Keep it simple
- Jr. engineers can manage
- Reuse existing systems
- As little code, config, as possible
- Fully automated
- Zero (low) false positives
- Low cost



# Inventory Management

## list all instances, running, PublicDnsName
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query "Reservations[*].Instances[*].PublicDnsName" \
  --output=text



# Access Management

## list all iam users, username
aws iam list-users --output text | cut -f 6

## list all system users
osqueryi --json "SELECT * FROM users"



# Config Management

## Cloud Resources
- WAF rules
- IAM User rules
- Security Group rules
- S3 bucket rules

## Server Instances
- SSH User rules
- Running processes
- Network services
- Service config

## list iam group rules
aws iam list-groups

aws iam list-attached-group-policies \
  --group-name <group_name>

## list all running processes
osqueryi --json "SELECT pid, name, cmdline FROM processes ORDER BY start_time ASC;"

## list all listening ports
osqueryi --json "SELECT * FROM listening_ports;"



# Patch management

## Ubuntu / Debian, list all installed packages
apt list --installed | cut -d' ' -f1,2 \
  > installed_packages.txt

## diff against default packages
create a new service instance, without anything installed to create the default_packages.txt file

grep -xvFf default_packages.txt installed_packages.txt

## list all installed packages
osqueryi --json "SELECT name, version, revision, FROM deb_packages;"



# Logging and Monitoring
Extract / Beats
Buffer / Kafka
Transform / Logstash
Load / ElasticSearch
View / Kibana



# Alerts
Load / Elasticsearch
Detect / ElastAlert
Alert / AWS SNS



# Automated Remediation

## lock down all open S3 buckets

https://gist.github.com/apolloclark/7f805c503a3b4427955cabe93c6d824b

aws s3api list-buckets \
  --query 'Buckets[*].[Name]' \
  --output text | \

xargs -I {} bash -c \

  'if [[ $(aws s3api get-bucket-acl --bucket {} --query '"'"'Grants[?Grantee.URI==`http://acs.amazonaws.com/groups/global/AllUsers` && Permission==`READ`]'"'"' --output text) ]]; \

  then \
    aws s3api put-bucket-acl --acl "private" --bucket {} ; \
  fi'



# Deployment Tools
Build VMs / Packer
Config Services / Ansible
Test Services / ServerSpec
Build Resources / Terraform


# Project Link

https://github.com/apolloclark/tf-aws
