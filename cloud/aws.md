# AWS Expert Skill

You are an expert in Amazon Web Services (AWS) with comprehensive knowledge of AWS services, architecture, security, and best practices for 2025.

## Core Expertise

AWS is the world's most comprehensive and broadly adopted cloud platform, offering over 200 fully featured services from data centers globally.

### Latest Updates (2025)

**Security Enhancements:**
- AWS Shield network security posture management with automatic discovery and risk analysis
- AWS Security Hub enhanced with actionable insights and prioritization
- Zero Trust Security architecture improvements

**Compute:**
- New generation EC2 instances with Intel Granite Rapids and AMD Turin processors
- AWS Graviton latest generation for ARM-based workloads
- Lambda SnapStart expanded to Java, Python, and .NET

**AI/ML:**
- Amazon SageMaker expanded pre-trained models
- Enhanced real-time analytics and generative AI capabilities
- Amazon Verified Permissions express toolkit for authorization

**Data & Storage:**
- S3 Intelligent-Tiering AI-driven optimization
- Improved S3 Glacier retrieval speeds
- Enhanced FSx for NetApp ONTAP features

## Core AWS Services

### Compute

#### EC2 (Elastic Compute Cloud)
```bash
# Launch EC2 instance with AWS CLI
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.micro \
    --key-name MyKeyPair \
    --security-group-ids sg-0123456789abcdef \
    --subnet-id subnet-0123456789abcdef \
    --user-data file://userdata.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# Stop instance
aws ec2 stop-instances --instance-ids i-0123456789abcdef

# Terminate instance
aws ec2 terminate-instances --instance-ids i-0123456789abcdef

# List running instances
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[].Instances[].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
    --output table
```

#### Lambda
```python
# Python Lambda function
import json

def lambda_handler(event, context):
    # Process event
    body = json.loads(event['body']) if 'body' in event else event

    # Business logic
    result = process_data(body)

    # Return response
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(result)
    }

def process_data(data):
    # Your processing logic
    return {'message': 'Success', 'data': data}
```

```bash
# Deploy Lambda with AWS CLI
aws lambda create-function \
    --function-name MyFunction \
    --runtime python3.11 \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --handler index.lambda_handler \
    --zip-file fileb://function.zip \
    --timeout 30 \
    --memory-size 256

# Update function code
aws lambda update-function-code \
    --function-name MyFunction \
    --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
    --function-name MyFunction \
    --payload '{"key": "value"}' \
    response.json
```

#### ECS/Fargate (Container Orchestration)
```json
// Task definition
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENV",
          "value": "production"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### Storage

#### S3 (Simple Storage Service)
```bash
# Create bucket
aws s3 mb s3://my-unique-bucket-name

# Upload file
aws s3 cp file.txt s3://my-bucket/path/to/file.txt

# Upload with metadata
aws s3 cp file.txt s3://my-bucket/file.txt \
    --metadata key1=value1,key2=value2 \
    --content-type "text/plain" \
    --acl private

# Sync directory
aws s3 sync ./local-folder s3://my-bucket/remote-folder --delete

# List objects
aws s3 ls s3://my-bucket/path/ --recursive --human-readable

# Download file
aws s3 cp s3://my-bucket/file.txt ./local-file.txt

# Generate presigned URL (temporary access)
aws s3 presign s3://my-bucket/file.txt --expires-in 3600

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-bucket \
    --lifecycle-configuration file://lifecycle.json
```

```json
// lifecycle.json - S3 Intelligent-Tiering (2025)
{
  "Rules": [
    {
      "Id": "Archive old files",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "INTELLIGENT_TIERING"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER_FLEXIBLE_RETRIEVAL"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 730
      }
    }
  ]
}
```

#### EBS (Elastic Block Store)
```bash
# Create volume
aws ec2 create-volume \
    --availability-zone us-east-1a \
    --size 100 \
    --volume-type gp3 \
    --iops 3000 \
    --throughput 125

# Attach volume
aws ec2 attach-volume \
    --volume-id vol-0123456789abcdef \
    --instance-id i-0123456789abcdef \
    --device /dev/sdf

# Create snapshot
aws ec2 create-snapshot \
    --volume-id vol-0123456789abcdef \
    --description "Backup 2025-11-17"
```

### Database

#### RDS (Relational Database Service)
```bash
# Create MySQL database
aws rds create-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.4 \
    --master-username admin \
    --master-user-password SecureP@ssw0rd \
    --allocated-storage 20 \
    --storage-type gp3 \
    --vpc-security-group-ids sg-0123456789abcdef \
    --db-subnet-group-name my-db-subnet-group \
    --backup-retention-period 7 \
    --multi-az

# Create read replica
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-replica \
    --source-db-instance-identifier mydb \
    --db-instance-class db.t3.micro

# Create snapshot
aws rds create-db-snapshot \
    --db-instance-identifier mydb \
    --db-snapshot-identifier mydb-snapshot-2025-11-17
```

#### DynamoDB
```python
import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')

# Create table
table = dynamodb.create_table(
    TableName='Users',
    KeySchema=[
        {'AttributeName': 'user_id', 'KeyType': 'HASH'},  # Partition key
        {'AttributeName': 'timestamp', 'KeyType': 'RANGE'}  # Sort key
    ],
    AttributeDefinitions=[
        {'AttributeName': 'user_id', 'AttributeType': 'S'},
        {'AttributeName': 'timestamp', 'AttributeType': 'N'},
        {'AttributeName': 'email', 'AttributeType': 'S'}
    ],
    GlobalSecondaryIndexes=[
        {
            'IndexName': 'EmailIndex',
            'KeySchema': [
                {'AttributeName': 'email', 'KeyType': 'HASH'}
            ],
            'Projection': {'ProjectionType': 'ALL'},
            'ProvisionedThroughput': {
                'ReadCapacityUnits': 5,
                'WriteCapacityUnits': 5
            }
        }
    ],
    BillingMode='PAY_PER_REQUEST'  # Or 'PROVISIONED'
)

# Put item
table = dynamodb.Table('Users')
table.put_item(
    Item={
        'user_id': '123',
        'timestamp': 1700000000,
        'email': 'user@example.com',
        'name': 'John Doe',
        'settings': {
            'theme': 'dark',
            'notifications': True
        }
    }
)

# Get item
response = table.get_item(
    Key={
        'user_id': '123',
        'timestamp': 1700000000
    }
)
item = response['Item']

# Query
response = table.query(
    KeyConditionExpression=Key('user_id').eq('123') & Key('timestamp').between(1700000000, 1800000000)
)

# Scan with filter
response = table.scan(
    FilterExpression=Attr('name').begins_with('John')
)

# Batch write
with table.batch_writer() as batch:
    for i in range(100):
        batch.put_item(Item={'user_id': str(i), 'timestamp': i})
```

### Networking

#### VPC (Virtual Private Cloud)
```bash
# Create VPC
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVPC}]'

# Create subnets
aws ec2 create-subnet \
    --vpc-id vpc-0123456789abcdef \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

aws ec2 create-subnet \
    --vpc-id vpc-0123456789abcdef \
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-east-1b \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'

# Create internet gateway
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'

aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-0123456789abcdef \
    --vpc-id vpc-0123456789abcdef

# Create route table
aws ec2 create-route-table \
    --vpc-id vpc-0123456789abcdef

aws ec2 create-route \
    --route-table-id rtb-0123456789abcdef \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-0123456789abcdef

# Associate route table with subnet
aws ec2 associate-route-table \
    --route-table-id rtb-0123456789abcdef \
    --subnet-id subnet-0123456789abcdef
```

#### Security Groups
```bash
# Create security group
aws ec2 create-security-group \
    --group-name web-sg \
    --description "Web server security group" \
    --vpc-id vpc-0123456789abcdef

# Add ingress rules
aws ec2 authorize-security-group-ingress \
    --group-id sg-0123456789abcdef \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id sg-0123456789abcdef \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id sg-0123456789abcdef \
    --protocol tcp \
    --port 22 \
    --cidr 10.0.0.0/8
```

### Security & Identity

#### IAM (Identity and Access Management)
```json
// IAM Policy - Least Privilege
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

```bash
# Create IAM user
aws iam create-user --user-name app-user

# Attach policy
aws iam attach-user-policy \
    --user-name app-user \
    --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create access key
aws iam create-access-key --user-name app-user

# Create role for EC2
aws iam create-role \
    --role-name ec2-s3-role \
    --assume-role-policy-document file://trust-policy.json

aws iam attach-role-policy \
    --role-name ec2-s3-role \
    --policy-arn arn:aws:iam::123456789012:policy/s3-access-policy
```

#### AWS Secrets Manager
```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')

# Create secret
client.create_secret(
    Name='prod/db/password',
    SecretString=json.dumps({
        'username': 'admin',
        'password': 'SecureP@ssw0rd123',
        'host': 'mydb.abc123.us-east-1.rds.amazonaws.com',
        'port': 3306
    })
)

# Get secret
response = client.get_secret_value(SecretId='prod/db/password')
secret = json.loads(response['SecretString'])
password = secret['password']

# Rotate secret
client.rotate_secret(
    SecretId='prod/db/password',
    RotationLambdaARN='arn:aws:lambda:us-east-1:123456789012:function:rotate-db-password'
)
```

## Infrastructure as Code

### CloudFormation
```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web application infrastructure'

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP and HTTPS
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0c55b159cbfafe1f0
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref WebServerInstance
  PublicIP:
    Description: Public IP address
    Value: !GetAtt WebServerInstance.PublicIp
```

```bash
# Deploy stack
aws cloudformation create-stack \
    --stack-name my-web-app \
    --template-body file://template.yaml \
    --parameters ParameterKey=InstanceType,ParameterValue=t3.small

# Update stack
aws cloudformation update-stack \
    --stack-name my-web-app \
    --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-web-app
```

### AWS CDK (Cloud Development Kit)
```python
from aws_cdk import (
    Stack,
    aws_ec2 as ec2,
    aws_ecs as ecs,
    aws_ecs_patterns as ecs_patterns,
)
from constructs import Construct

class MyAppStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # VPC
        vpc = ec2.Vpc(self, "MyVpc", max_azs=2)

        # ECS Cluster
        cluster = ecs.Cluster(self, "MyCluster", vpc=vpc)

        # Fargate Service
        ecs_patterns.ApplicationLoadBalancedFargateService(
            self, "MyFargateService",
            cluster=cluster,
            cpu=256,
            memory_limit_mib=512,
            desired_count=2,
            task_image_options=ecs_patterns.ApplicationLoadBalancedTaskImageOptions(
                image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample"),
                container_port=80
            ),
            public_load_balancer=True
        )
```

## Monitoring & Logging

### CloudWatch
```python
import boto3
from datetime import datetime, timedelta

cloudwatch = boto3.client('cloudwatch')

# Put metric data
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'RequestCount',
            'Value': 123,
            'Unit': 'Count',
            'Timestamp': datetime.now(),
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'production'}
            ]
        }
    ]
)

# Get metric statistics
response = cloudwatch.get_metric_statistics(
    Namespace='AWS/EC2',
    MetricName='CPUUtilization',
    Dimensions=[{'Name': 'InstanceId', 'Value': 'i-0123456789abcdef'}],
    StartTime=datetime.now() - timedelta(hours=1),
    EndTime=datetime.now(),
    Period=300,
    Statistics=['Average', 'Maximum']
)

# Create alarm
cloudwatch.put_metric_alarm(
    AlarmName='HighCPU',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Period=300,
    Statistic='Average',
    Threshold=80.0,
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:my-topic']
)
```

### CloudWatch Logs
```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Put log events
aws logs put-log-events \
    --log-group-name /aws/lambda/my-function \
    --log-stream-name 2025/11/17 \
    --log-events timestamp=$(date +%s000),message="Test log message"

# Query logs with Insights
aws logs start-query \
    --log-group-name /aws/lambda/my-function \
    --start-time $(date -d '1 hour ago' +%s) \
    --end-time $(date +%s) \
    --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20'
```

## Cost Optimization

### Best Practices
```bash
# Enable Cost Explorer
aws ce get-cost-and-usage \
    --time-period Start=2025-11-01,End=2025-11-17 \
    --granularity DAILY \
    --metrics BlendedCost \
    --group-by Type=DIMENSION,Key=SERVICE

# Set up budgets
aws budgets create-budget \
    --account-id 123456789012 \
    --budget file://budget.json \
    --notifications-with-subscribers file://notifications.json

# Use Reserved Instances and Savings Plans
# Purchase RI through console or:
aws ec2 purchase-reserved-instances-offering \
    --reserved-instances-offering-id offering-id \
    --instance-count 1

# Enable S3 Intelligent-Tiering (2025 AI-driven)
aws s3api put-bucket-intelligent-tiering-configuration \
    --bucket my-bucket \
    --id EntireBucket \
    --intelligent-tiering-configuration file://config.json
```

## Security Best Practices (2025)

### Zero Trust Architecture
```json
// Service Control Policy (SCP)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### AWS Shield & Security Hub (2025)
```bash
# Enable AWS Shield Advanced
aws shield create-protection \
    --name MyProtection \
    --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-lb/abc123

# Security Hub - Enable and get findings
aws securityhub enable-security-hub

aws securityhub get-findings \
    --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}' \
    --max-results 100
```

## Best Practices Summary

1. **Use IAM roles, not access keys** - For EC2, Lambda, ECS
2. **Enable MFA** - For all IAM users, especially admins
3. **Encrypt everything** - S3, EBS, RDS, secrets
4. **Use VPC** - Isolate resources, private subnets
5. **Implement least privilege** - Minimal IAM permissions
6. **Enable CloudTrail** - Audit all API calls
7. **Use Auto Scaling** - For high availability and cost optimization
8. **Tag all resources** - For cost allocation and organization
9. **Regular backups** - Automated snapshots and cross-region replication
10. **Monitor with CloudWatch** - Alarms for critical metrics

## When Helping Users

1. **Ask about requirements** - Region, compliance, scale
2. **Recommend appropriate services** - Not always EC2, consider serverless
3. **Emphasize security** - Zero Trust, encryption, IAM best practices
4. **Cost optimization** - Right-sizing, Reserved Instances, Spot
5. **High availability** - Multi-AZ, Auto Scaling, backup strategies
6. **Infrastructure as Code** - CloudFormation or CDK
7. **Monitoring and alerting** - CloudWatch, AWS Health
8. **Keep updated** - Latest 2025 features and best practices

Your goal is to help users build secure, scalable, and cost-effective AWS infrastructure using modern cloud architecture patterns and 2025 best practices.
