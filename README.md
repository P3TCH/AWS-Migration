# AWS Account Migration Plan

## Overview
This document outlines the plan to migrate AWS resources from the old AWS account to the new AWS account. The services being used that need to be migrated include:

- EC2
- DynamoDB
- ElastiCache
- CodeCommit
- Route53
- ACM
- CloudFront
- WAF
- S3
- CloudWatch

## Prerequisites
1. Ensure you have administrative access to both the old and new AWS accounts.
2. Set up the new AWS account with appropriate IAM roles and permissions.
3. Install and configure the AWS CLI for both accounts.

## Migration Steps

### 1. EC2
- **AMI Creation**: Create AMIs for all EC2 instances in the old account.
  - `aws ec2 create-image --instance-id <instance-id> --name "<image-name>" --no-reboot`
- **Copy AMIs**: Copy the AMIs to the new account.
  - `aws ec2 copy-image --source-image-id <ami-id> --source-region <region> --region <new-region> --name "<new-image-name>" --source-account-id <old-account-id>`
- **Launch Instances**: Launch new instances in the new account using the copied AMIs.
- **Data Transfer**: Transfer data from the old instances to the new instances as needed.

### 2. DynamoDB
- **Export Data**: Export DynamoDB tables to S3.
  - `aws dynamodb export-table-to-point-in-time --table-arn <table-arn> --s3-bucket <bucket-name> --s3-prefix <prefix>`
- **Import Data**: Import the exported data into DynamoDB tables in the new account.
  - Use AWS Glue or DynamoDB import feature to import the data.

### 3. ElastiCache
- **Snapshot Creation**: Create snapshots of ElastiCache clusters.
  - `aws elasticache create-snapshot --cache-cluster-id <cluster-id> --snapshot-name <snapshot-name>`
- **Copy Snapshots**: Copy the snapshots to the new account.
  - `aws elasticache copy-snapshot --source-snapshot-name <snapshot-name> --target-snapshot-name <new-snapshot-name> --target-bucket <bucket-name>`
- **Restore Snapshots**: Restore the snapshots in the new account.
  - `aws elasticache create-cache-cluster --snapshot-name <new-snapshot-name> --cache-cluster-id <new-cluster-id>`

### 4. CodeCommit
- **Repository Clone**: Clone the repositories from the old account.
  - `git clone <old-repo-url>`
- **Create Repositories**: Create new repositories in the new account.
  - `aws codecommit create-repository --repository-name <new-repo-name>`
- **Push Repositories**: Push the cloned repositories to the new account.
  - `git remote add new-origin <new-repo-url>`
  - `git push new-origin --all`

### 5. Route53
- **Export Zones**: Export hosted zones from the old account.
  - Use AWS CLI or third-party tools to export the zone files.
- **Import Zones**: Import the hosted zones into the new account.
  - `aws route53 create-hosted-zone --name <domain-name> --caller-reference <unique-string>`

### 6. ACM
- **Certificate Export**: Export certificates from ACM in the old account.
  - Use the AWS Management Console to export the certificates.
- **Certificate Import**: Import the certificates into ACM in the new account.
  - `aws acm import-certificate --certificate fileb://<cert-file> --private-key fileb://<key-file> --certificate-chain fileb://<chain-file>`

### 7. CloudFront
- **Distribution Export**: Export CloudFront distribution configurations from the old account.
  - Use the AWS Management Console or AWS CLI to export configurations.
- **Distribution Creation**: Create new CloudFront distributions in the new account using the exported configurations.
  - `aws cloudfront create-distribution --distribution-config file://<config-file>`

### 8. WAF
- **WebACL Export**: Export WAF WebACL configurations from the old account.
  - Use the AWS Management Console or AWS CLI to export configurations.
- **WebACL Creation**: Create new WebACLs in the new account using the exported configurations.
  - `aws wafv2 create-web-acl --scope <scope> --name <name> --region <region> --rules file://<rules-file> --visibility-config <visibility-config>`

### 9. S3
- **Bucket Replication**: Set up cross-account replication for S3 buckets.
  - Configure replication rules in the source bucket to replicate objects to the target bucket in the new account.
  - `aws s3api put-bucket-replication --bucket <source-bucket> --replication-configuration file://<replication-config-file>`
- **Manual Copy**: Alternatively, manually copy objects between buckets using the AWS CLI.
  - `aws s3 sync s3://<source-bucket> s3://<target-bucket>`

### 10. CloudWatch
- **Metric Export**: Export CloudWatch metrics from the old account.
  - Use the AWS Management Console or AWS CLI to export metrics.
- **Metric Import**: Import the metrics into CloudWatch in the new account.
  - Use third-party tools or AWS SDKs to import metrics.

## Post-Migration
1. Verify all resources and configurations in the new account.
2. Update any application configurations or environment variables to point to the new resources.
3. Monitor the new environment to ensure everything is functioning as expected.
4. Decommission resources in the old account once the migration is confirmed successful.

## Rollback Plan
In case of any issues during the migration, the following rollback plan should be executed:
1. Revert DNS changes in Route53 to point to the old account.
2. Restore services in the old account from the latest backups and snapshots.
3. Notify stakeholders about the rollback and the next steps.

## Conclusion
This migration plan provides a comprehensive approach to migrating AWS resources from one account to another. Ensure to follow each step carefully and validate the migration at each stage to minimize downtime and data loss.
