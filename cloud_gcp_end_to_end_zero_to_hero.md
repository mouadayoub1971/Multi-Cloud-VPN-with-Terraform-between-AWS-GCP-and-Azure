# Google Cloud Platform (GCP) - Zero to Hero Guide for AWS Experts

## Table of Contents
1. [Introduction & Core Concepts](#introduction--core-concepts)
2. [Account & Organization Structure](#account--organization-structure)
3. [Identity & Access Management (IAM)](#identity--access-management-iam)
4. [Compute Services](#compute-services)
5. [Storage Services](#storage-services)
6. [Database Services](#database-services)
7. [Networking](#networking)
8. [Container & Kubernetes Services](#container--kubernetes-services)
9. [Serverless & Functions](#serverless--functions)
10. [Big Data & Analytics](#big-data--analytics)
11. [Machine Learning & AI](#machine-learning--ai)
12. [Security & Compliance](#security--compliance)
13. [Monitoring & Logging](#monitoring--logging)
14. [DevOps & CI/CD](#devops--cicd)
15. [Cost Management](#cost-management)
16. [Best Practices & Patterns](#best-practices--patterns)

---

## Introduction & Core Concepts

### GCP vs AWS: Fundamental Differences

| Concept | AWS | GCP |
|---------|-----|-----|
| **Global Infrastructure** | Regions → Availability Zones | Regions → Zones |
| **Resource Hierarchy** | Organizations → Accounts → Resources | Organization → Folders → Projects → Resources |
| **CLI Tool** | AWS CLI | gcloud CLI |
| **Infrastructure as Code** | CloudFormation | Deployment Manager (or Terraform) |
| **Console** | AWS Management Console | Google Cloud Console |

### Key Terminology Mapping

| AWS Term | GCP Equivalent | Description |
|----------|----------------|-------------|
| Account | Project | Logical container for resources |
| AWS Organizations | Google Cloud Organization | Multi-project/account management |
| Availability Zone (AZ) | Zone | Isolated location within a region |
| Region | Region | Geographic location |
| IAM Role | Service Account | Identity for services/applications |
| EC2 | Compute Engine (GCE) | Virtual machines |
| S3 | Cloud Storage (GCS) | Object storage |
| Lambda | Cloud Functions / Cloud Run | Serverless compute |
| RDS | Cloud SQL | Managed relational databases |
| DynamoDB | Firestore / Bigtable | NoSQL databases |
| ECS/EKS | GKE (Google Kubernetes Engine) | Container orchestration |
| CloudWatch | Cloud Monitoring (formerly Stackdriver) | Monitoring and logging |
| VPC | VPC (same name!) | Virtual private cloud |
| Route 53 | Cloud DNS | DNS service |
| CloudFront | Cloud CDN | Content delivery network |
| KMS | Cloud KMS | Key management service |

---

## Account & Organization Structure

### GCP Resource Hierarchy

```
Organization (example.com)
  ├── Folder: Production
  │     ├── Project: prod-frontend
  │     ├── Project: prod-backend
  │     └── Project: prod-data
  ├── Folder: Development
  │     ├── Project: dev-frontend
  │     └── Project: dev-backend
  └── Folder: Shared Services
        ├── Project: shared-networking
        └── Project: shared-security
```

**AWS Equivalent:**
```
AWS Organization
  ├── OU: Production
  │     ├── Account: prod-frontend
  │     ├── Account: prod-backend
  │     └── Account: prod-data
  └── OU: Development
```

### Creating a Project (CLI)

**GCP:**
```bash
# Set up gcloud CLI
gcloud init

# Create a new project
gcloud projects create my-project-id \
  --name="My Project" \
  --organization=123456789

# Set the current project
gcloud config set project my-project-id

# List all projects
gcloud projects list

# Enable billing for project
gcloud billing projects link my-project-id \
  --billing-account=0X0X0X-0X0X0X-0X0X0X
```

**AWS Equivalent:**
```bash
# Create a new account (requires AWS Organizations)
aws organizations create-account \
  --email prod@example.com \
  --account-name "Production Account"
```

### Enabling APIs

In GCP, you must explicitly enable APIs for each project:

```bash
# Enable Compute Engine API
gcloud services enable compute.googleapis.com

# Enable Cloud Storage API
gcloud services enable storage.googleapis.com

# Enable multiple APIs at once
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  sql-component.googleapis.com \
  container.googleapis.com

# List enabled services
gcloud services list --enabled

# List available services
gcloud services list --available
```

**AWS Note:** In AWS, most services are available by default without explicit enablement.

---

## Identity & Access Management (IAM)

### IAM Concepts

**GCP has three types of IAM members:**
1. **Google Account** - individual user (you@gmail.com)
2. **Service Account** - application/service identity
3. **Google Group** - collection of users
4. **Google Workspace/Cloud Identity domain** - all users in org

### Predefined Roles vs Custom Roles

**GCP:**
```bash
# Grant a user viewer role on a project
gcloud projects add-iam-policy-binding my-project-id \
  --member="user:alice@example.com" \
  --role="roles/viewer"

# Grant service account storage admin role
gcloud projects add-iam-policy-binding my-project-id \
  --member="serviceAccount:my-sa@my-project-id.iam.gserviceaccount.com" \
  --role="roles/storage.admin"

# View IAM policy for a project
gcloud projects get-iam-policy my-project-id

# Create a custom role
gcloud iam roles create myCustomRole \
  --project=my-project-id \
  --title="My Custom Role" \
  --description="Custom role for specific permissions" \
  --permissions=compute.instances.get,compute.instances.list \
  --stage=ALPHA
```

**AWS Equivalent:**
```bash
# Attach managed policy to user
aws iam attach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create custom policy
aws iam create-policy \
  --policy-name MyCustomPolicy \
  --policy-document file://policy.json
```

### Service Accounts (≈ IAM Roles in AWS)

**GCP:**
```bash
# Create a service account
gcloud iam service-accounts create my-service-account \
  --display-name="My Service Account" \
  --description="Service account for app"

# Grant service account permissions
gcloud projects add-iam-policy-binding my-project-id \
  --member="serviceAccount:my-service-account@my-project-id.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Create and download service account key (JSON)
gcloud iam service-accounts keys create ~/key.json \
  --iam-account=my-service-account@my-project-id.iam.gserviceaccount.com

# List service accounts
gcloud iam service-accounts list

# Allow a user to impersonate a service account
gcloud iam service-accounts add-iam-policy-binding \
  my-service-account@my-project-id.iam.gserviceaccount.com \
  --member="user:alice@example.com" \
  --role="roles/iam.serviceAccountUser"
```

**AWS Equivalent:**
```bash
# Create IAM role
aws iam create-role \
  --role-name MyServiceRole \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name MyServiceRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### IAM Best Practices

| Practice | GCP Implementation | AWS Equivalent |
|----------|-------------------|----------------|
| **Least Privilege** | Use predefined roles, avoid primitive roles (Owner, Editor, Viewer) | Use managed policies, avoid AdministratorAccess |
| **Service Identity** | Use service accounts | Use IAM roles for EC2/Lambda |
| **Temporary Credentials** | Service account impersonation | AssumeRole |
| **MFA** | Enforce 2-Step Verification in Cloud Identity | Enforce MFA via IAM policies |

---

## Compute Services

### Compute Engine (GCE) - Virtual Machines

**GCP:**
```bash
# Create a VM instance
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --boot-disk-size=10GB \
  --boot-disk-type=pd-standard \
  --tags=web-server,http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx'

# List instances
gcloud compute instances list

# SSH into instance
gcloud compute ssh my-vm --zone=us-central1-a

# Stop instance
gcloud compute instances stop my-vm --zone=us-central1-a

# Start instance
gcloud compute instances start my-vm --zone=us-central1-a

# Delete instance
gcloud compute instances delete my-vm --zone=us-central1-a

# Create instance with custom service account
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --service-account=my-sa@my-project-id.iam.gserviceaccount.com \
  --scopes=https://www.googleapis.com/auth/cloud-platform
```

**AWS Equivalent:**
```bash
# Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-12345678 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --user-data file://startup-script.sh \
  --iam-instance-profile Name=MyInstanceRole
```

### Machine Types & Pricing

**GCP Machine Families:**
- **E2** - Cost-optimized (AWS: T3, T4g)
- **N2/N2D** - Balanced (AWS: M5, M6i)
- **C2/C2D** - Compute-optimized (AWS: C5, C6i)
- **M2** - Memory-optimized (AWS: R5, R6i)
- **A2** - GPU instances (AWS: P3, P4)

**Example Machine Types:**
```bash
# List available machine types in a zone
gcloud compute machine-types list --filter="zone:us-central1-a"

# Common machine types:
# e2-micro      (0.25 vCPU, 1 GB) - Free tier eligible
# e2-small      (0.5 vCPU, 2 GB)
# e2-medium     (1 vCPU, 4 GB)
# e2-standard-2 (2 vCPU, 8 GB)
# n2-standard-4 (4 vCPU, 16 GB)
# c2-standard-8 (8 vCPU, 32 GB)
```

### Preemptible VMs (Spot Instances)

**GCP:**
```bash
# Create preemptible VM (up to 91% cheaper)
gcloud compute instances create preemptible-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --preemptible \
  --maintenance-policy=TERMINATE

# Create Spot VM (newer version, more flexible)
gcloud compute instances create spot-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP
```

**AWS Equivalent:**
```bash
# Launch Spot instance
aws ec2 request-spot-instances \
  --instance-count 1 \
  --type "one-time" \
  --launch-specification file://specification.json
```

### Instance Templates & Managed Instance Groups

**GCP:**
```bash
# Create instance template
gcloud compute instance-templates create web-server-template \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --tags=web-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx'

# Create managed instance group (MIG)
gcloud compute instance-groups managed create web-server-group \
  --base-instance-name=web-server \
  --template=web-server-template \
  --size=3 \
  --zone=us-central1-a

# Set autoscaling
gcloud compute instance-groups managed set-autoscaling web-server-group \
  --zone=us-central1-a \
  --max-num-replicas=10 \
  --min-num-replicas=2 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=90

# Create regional MIG (multi-zone)
gcloud compute instance-groups managed create web-server-regional \
  --base-instance-name=web-server \
  --template=web-server-template \
  --size=6 \
  --region=us-central1 \
  --zones=us-central1-a,us-central1-b,us-central1-c
```

**AWS Equivalent:**
```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name web-server-template \
  --launch-template-data file://template.json

# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name web-server-asg \
  --launch-template LaunchTemplateName=web-server-template \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3 \
  --availability-zones us-east-1a us-east-1b
```

---

## Storage Services

### Cloud Storage (GCS) - Object Storage

**Storage Classes:**

| GCP Class | AWS Equivalent | Use Case | Min Storage Duration |
|-----------|----------------|----------|---------------------|
| Standard | S3 Standard | Frequently accessed | None |
| Nearline | S3 Standard-IA | Once per month | 30 days |
| Coldline | S3 Glacier Instant Retrieval | Once per quarter | 90 days |
| Archive | S3 Glacier Deep Archive | Once per year | 365 days |

**GCP:**
```bash
# Create a bucket
gsutil mb -p my-project-id -c STANDARD -l us-central1 gs://my-unique-bucket-name/

# Upload a file
gsutil cp myfile.txt gs://my-unique-bucket-name/

# Upload directory recursively
gsutil cp -r ./my-directory gs://my-unique-bucket-name/

# List buckets
gsutil ls

# List objects in bucket
gsutil ls gs://my-unique-bucket-name/

# Download file
gsutil cp gs://my-unique-bucket-name/myfile.txt ./

# Make object public
gsutil acl ch -u AllUsers:R gs://my-unique-bucket-name/myfile.txt

# Set bucket lifecycle (auto-delete after 30 days)
cat > lifecycle.json << EOF
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "Delete"},
        "condition": {"age": 30}
      }
    ]
  }
}
EOF
gsutil lifecycle set lifecycle.json gs://my-unique-bucket-name/

# Enable versioning
gsutil versioning set on gs://my-unique-bucket-name/

# Sync directory to bucket
gsutil rsync -r ./local-dir gs://my-unique-bucket-name/remote-dir

# Set storage class
gsutil rewrite -s NEARLINE gs://my-unique-bucket-name/myfile.txt
```

**AWS Equivalent:**
```bash
# Create bucket
aws s3 mb s3://my-bucket-name --region us-east-1

# Upload file
aws s3 cp myfile.txt s3://my-bucket-name/

# Sync directory
aws s3 sync ./local-dir s3://my-bucket-name/remote-dir/

# List buckets
aws s3 ls

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled
```

### Persistent Disks (EBS)

**GCP:**
```bash
# Create a persistent disk
gcloud compute disks create my-disk \
  --size=100GB \
  --type=pd-standard \
  --zone=us-central1-a

# Create SSD persistent disk
gcloud compute disks create my-ssd-disk \
  --size=100GB \
  --type=pd-ssd \
  --zone=us-central1-a

# Attach disk to instance
gcloud compute instances attach-disk my-vm \
  --disk=my-disk \
  --zone=us-central1-a

# Create snapshot
gcloud compute disks snapshot my-disk \
  --snapshot-names=my-snapshot \
  --zone=us-central1-a

# Create disk from snapshot
gcloud compute disks create restored-disk \
  --source-snapshot=my-snapshot \
  --zone=us-central1-a

# Resize disk
gcloud compute disks resize my-disk \
  --size=200GB \
  --zone=us-central1-a
```

**Disk Types:**
- **pd-standard** - HDD (AWS: gp2)
- **pd-balanced** - SSD balanced performance/cost (AWS: gp3)
- **pd-ssd** - High-performance SSD (AWS: io1/io2)
- **pd-extreme** - Highest performance (AWS: io2 Block Express)

**AWS Equivalent:**
```bash
# Create EBS volume
aws ec2 create-volume \
  --size 100 \
  --volume-type gp3 \
  --availability-zone us-east-1a

# Attach volume
aws ec2 attach-volume \
  --volume-id vol-12345678 \
  --instance-id i-12345678 \
  --device /dev/sdf

# Create snapshot
aws ec2 create-snapshot --volume-id vol-12345678
```

### Filestore (EFS)

**GCP:**
```bash
# Create Filestore instance (NFS)
gcloud filestore instances create my-filestore \
  --zone=us-central1-a \
  --tier=BASIC_HDD \
  --file-share=name=share1,capacity=1TB \
  --network=name=default

# List instances
gcloud filestore instances list

# Mount on VM:
# sudo apt-get install nfs-common
# sudo mkdir /mnt/filestore
# sudo mount <FILESTORE_IP>:/share1 /mnt/filestore
```

**AWS Equivalent:**
```bash
# Create EFS file system
aws efs create-file-system \
  --performance-mode generalPurpose \
  --encrypted
```

---

## Database Services

### Cloud SQL (RDS)

**Supported Databases:**
- MySQL
- PostgreSQL
- SQL Server

**GCP:**
```bash
# Create MySQL instance
gcloud sql instances create my-mysql-instance \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-1 \
  --region=us-central1 \
  --root-password=mypassword \
  --storage-type=SSD \
  --storage-size=10GB \
  --backup-start-time=03:00 \
  --enable-bin-log \
  --availability-type=REGIONAL

# Create PostgreSQL instance
gcloud sql instances create my-postgres-instance \
  --database-version=POSTGRES_14 \
  --tier=db-custom-2-7680 \
  --region=us-central1

# Create database
gcloud sql databases create mydatabase \
  --instance=my-mysql-instance

# Create user
gcloud sql users create myuser \
  --instance=my-mysql-instance \
  --password=mypassword

# Connect to instance
gcloud sql connect my-mysql-instance --user=root

# Create read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-mysql-instance \
  --tier=db-n1-standard-1 \
  --region=us-east1

# Enable point-in-time recovery
gcloud sql instances patch my-mysql-instance \
  --enable-bin-log \
  --backup-start-time=02:00

# List instances
gcloud sql instances list
```

**AWS Equivalent:**
```bash
# Create RDS MySQL instance
aws rds create-db-instance \
  --db-instance-identifier my-mysql-instance \
  --db-instance-class db.t3.medium \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password mypassword \
  --allocated-storage 20 \
  --storage-type gp3 \
  --multi-az

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-replica \
  --source-db-instance-identifier my-mysql-instance
```

### Firestore (NoSQL Document DB)

**GCP:**
```bash
# Create Firestore database (done via console or programmatically)
gcloud firestore databases create --region=us-central

# Using Python SDK:
```python
from google.cloud import firestore

# Initialize Firestore client
db = firestore.Client()

# Create document
doc_ref = db.collection('users').document('user1')
doc_ref.set({
    'name': 'John Doe',
    'email': 'john@example.com',
    'age': 30
})

# Read document
doc = db.collection('users').document('user1').get()
print(doc.to_dict())

# Query documents
users = db.collection('users').where('age', '>', 25).stream()
for user in users:
    print(f'{user.id} => {user.to_dict()}')

# Update document
doc_ref.update({
    'age': 31
})

# Delete document
doc_ref.delete()

# Batch write
batch = db.batch()
doc_ref1 = db.collection('users').document('user2')
batch.set(doc_ref1, {'name': 'Jane'})
doc_ref2 = db.collection('users').document('user3')
batch.set(doc_ref2, {'name': 'Bob'})
batch.commit()

# Transaction
@firestore.transactional
def update_balance(transaction, account_ref, amount):
    snapshot = account_ref.get(transaction=transaction)
    balance = snapshot.get('balance')
    transaction.update(account_ref, {'balance': balance + amount})

transaction = db.transaction()
account_ref = db.collection('accounts').document('account1')
update_balance(transaction, account_ref, 100)
```

**AWS Equivalent (DynamoDB):**
```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('users')

# Put item
table.put_item(
    Item={
        'userId': 'user1',
        'name': 'John Doe',
        'email': 'john@example.com',
        'age': 30
    }
)

# Get item
response = table.get_item(Key={'userId': 'user1'})
print(response['Item'])

# Query
response = table.query(
    KeyConditionExpression='userId = :userId',
    ExpressionAttributeValues={':userId': 'user1'}
)
```

### Cloud Bigtable (NoSQL Wide-Column)

**Best for:** Time-series data, IoT, financial data (similar to AWS DynamoDB or Cassandra)

**GCP:**
```bash
# Create Bigtable instance
gcloud bigtable instances create my-bigtable-instance \
  --display-name="My Bigtable Instance" \
  --cluster-config=id=my-cluster,zone=us-central1-a,nodes=3 \
  --instance-type=PRODUCTION

# Create table (via cbt CLI)
cbt -project=my-project-id -instance=my-bigtable-instance createtable my-table

# Create column family
cbt -project=my-project-id -instance=my-bigtable-instance createfamily my-table cf1

# Write data
cbt -project=my-project-id -instance=my-bigtable-instance set my-table row1 cf1:col1=value1

# Read data
cbt -project=my-project-id -instance=my-bigtable-instance read my-table
```

**Using Python:**
```python
from google.cloud import bigtable
from google.cloud.bigtable import column_family, row_filters

client = bigtable.Client(project='my-project-id', admin=True)
instance = client.instance('my-bigtable-instance')
table = instance.table('my-table')

# Write data
row_key = 'user#123#2024-01-01'.encode()
row = table.direct_row(row_key)
row.set_cell('cf1', 'name', 'John Doe', timestamp=datetime.datetime.utcnow())
row.set_cell('cf1', 'email', 'john@example.com', timestamp=datetime.datetime.utcnow())
row.commit()

# Read data
row = table.read_row(row_key)
print(row.cells['cf1'][b'name'][0].value.decode('utf-8'))

# Scan rows
rows = table.read_rows()
for row in rows:
    print(row.row_key.decode('utf-8'))
```

### Cloud Spanner (Globally Distributed SQL)

**GCP's unique offering** - globally distributed, horizontally scalable SQL database with strong consistency

**GCP:**
```bash
# Create Spanner instance
gcloud spanner instances create my-spanner-instance \
  --config=regional-us-central1 \
  --description="My Spanner Instance" \
  --nodes=1

# Create database
gcloud spanner databases create my-database \
  --instance=my-spanner-instance \
  --ddl='CREATE TABLE Users (
    UserId INT64 NOT NULL,
    Name STRING(100),
    Email STRING(100)
  ) PRIMARY KEY (UserId)'

# Query via Python:
```python
from google.cloud import spanner

client = spanner.Client()
instance = client.instance('my-spanner-instance')
database = instance.database('my-database')

# Insert data
def insert_user(transaction):
    transaction.execute_update(
        "INSERT Users (UserId, Name, Email) VALUES (1, 'John', 'john@example.com')"
    )

database.run_in_transaction(insert_user)

# Query data
with database.snapshot() as snapshot:
    results = snapshot.execute_sql("SELECT * FROM Users WHERE UserId = 1")
    for row in results:
        print(row)
```

**AWS Note:** Aurora Global Database is the closest equivalent, but Spanner offers stronger consistency guarantees across regions.

---

## Networking

### VPC (Virtual Private Cloud)

**GCP VPC Differences from AWS:**
- VPCs are **global** resources (not regional)
- Subnets are **regional** (span all zones in a region)
- One default VPC per project
- Shared VPC allows resource sharing across projects

**GCP:**
```bash
# Create custom VPC
gcloud compute networks create my-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional

# Create subnet (regional - spans all zones)
gcloud compute networks subnets create my-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

# Create subnet with secondary IP ranges (for GKE pods/services)
gcloud compute networks subnets create my-gke-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24 \
  --secondary-range=pods=10.1.0.0/16,services=10.2.0.0/16

# List VPCs
gcloud compute networks list

# List subnets
gcloud compute networks subnets list

# Delete VPC (must delete all resources first)
gcloud compute networks delete my-vpc
```

**AWS Equivalent:**
```bash
# Create VPC (regional resource)
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet (zonal)
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a
```

### Firewall Rules (Security Groups)

**GCP:**
```bash
# Create firewall rule (allow HTTP from anywhere)
gcloud compute firewall-rules create allow-http \
  --network=my-vpc \
  --allow=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# Allow HTTPS
gcloud compute firewall-rules create allow-https \
  --network=my-vpc \
  --allow=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# Allow SSH from specific IP
gcloud compute firewall-rules create allow-ssh \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=203.0.113.0/24

# Allow internal traffic
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --allow=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/8

# Deny rule (priority - lower number = higher priority)
gcloud compute firewall-rules create deny-telnet \
  --network=my-vpc \
  --action=DENY \
  --rules=tcp:23 \
  --source-ranges=0.0.0.0/0 \
  --priority=100

# List firewall rules
gcloud compute firewall-rules list

# Update firewall rule
gcloud compute firewall-rules update allow-http \
  --source-ranges=192.0.2.0/24

# Delete firewall rule
gcloud compute firewall-rules delete allow-http
```

**Key Concepts:**
- **target-tags**: Apply to instances with matching network tags
- **target-service-accounts**: Apply to instances using specific service accounts
- **source-tags**: Traffic from instances with specific tags
- **source-service-accounts**: Traffic from specific service accounts
- **priority**: 0-65535 (lower = higher priority, default = 1000)

**AWS Equivalent:**
```bash
# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-12345678

# Add ingress rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Cloud Load Balancing

**GCP has several load balancer types:**

| Type | Layer | Scope | AWS Equivalent |
|------|-------|-------|----------------|
| HTTP(S) Load Balancer | L7 | Global | CloudFront + ALB |
| TCP/UDP Load Balancer | L4 | Regional | NLB |
| Internal HTTP(S) Load Balancer | L7 | Regional | Internal ALB |
| Internal TCP/UDP Load Balancer | L4 | Regional | Internal NLB |

**Create HTTP(S) Load Balancer:**

```bash
# 1. Create instance group (done earlier)
# 2. Create health check
gcloud compute health-checks create http http-basic-check \
  --port=80 \
  --request-path=/health \
  --check-interval=10s \
  --timeout=5s \
  --healthy-threshold=2 \
  --unhealthy-threshold=3

# 3. Create backend service
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --health-checks=http-basic-check \
  --global

# 4. Add instance group to backend service
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=web-server-group \
  --instance-group-zone=us-central1-a \
  --global

# 5. Create URL map
gcloud compute url-maps create web-map \
  --default-service=web-backend-service

# 6. Create target HTTP proxy
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map=web-map

# 7. Create forwarding rule (frontend)
gcloud compute forwarding-rules create http-content-rule \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80

# Get the external IP
gcloud compute forwarding-rules describe http-content-rule --global

# For HTTPS, create SSL certificate:
gcloud compute ssl-certificates create my-cert \
  --certificate=cert.pem \
  --private-key=key.pem

# Create target HTTPS proxy
gcloud compute target-https-proxies create https-lb-proxy \
  --url-map=web-map \
  --ssl-certificates=my-cert

# Create HTTPS forwarding rule
gcloud compute forwarding-rules create https-content-rule \
  --global \
  --target-https-proxy=https-lb-proxy \
  --ports=443
```

**Create Network Load Balancer (TCP/UDP):**

```bash
# Create health check
gcloud compute health-checks create tcp tcp-health-check \
  --port=80

# Create backend service
gcloud compute backend-services create tcp-backend \
  --protocol=TCP \
  --health-checks=tcp-health-check \
  --region=us-central1

# Add backend
gcloud compute backend-services add-backend tcp-backend \
  --instance-group=web-server-group \
  --instance-group-zone=us-central1-a \
  --region=us-central1

# Create forwarding rule
gcloud compute forwarding-rules create tcp-lb \
  --region=us-central1 \
  --ports=80 \
  --backend-service=tcp-backend
```

### Cloud CDN

**GCP:**
```bash
# Enable Cloud CDN on backend service
gcloud compute backend-services update web-backend-service \
  --enable-cdn \
  --global

# Set cache mode
gcloud compute backend-services update web-backend-service \
  --cache-mode=CACHE_ALL_STATIC \
  --global

# Invalidate cache
gcloud compute url-maps invalidate-cdn-cache web-map \
  --path="/static/*" \
  --global
```

**AWS Equivalent:**
```bash
# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name my-bucket.s3.amazonaws.com \
  --default-root-object index.html
```

### Cloud DNS

**GCP:**
```bash
# Create managed zone
gcloud dns managed-zones create my-zone \
  --dns-name="example.com." \
  --description="My DNS zone"

# Start transaction
gcloud dns record-sets transaction start --zone=my-zone

# Add A record
gcloud dns record-sets transaction add "35.201.123.45" \
  --name="www.example.com." \
  --ttl=300 \
  --type=A \
  --zone=my-zone

# Add CNAME record
gcloud dns record-sets transaction add "www.example.com." \
  --name="blog.example.com." \
  --ttl=300 \
  --type=CNAME \
  --zone=my-zone

# Execute transaction
gcloud dns record-sets transaction execute --zone=my-zone

# List records
gcloud dns record-sets list --zone=my-zone
```

**AWS Equivalent:**
```bash
# Create hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference 2024-01-01

# Create A record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch file://change-batch.json
```

### Cloud VPN

**GCP:**
```bash
# Create VPN gateway
gcloud compute target-vpn-gateways create my-vpn-gateway \
  --network=my-vpc \
  --region=us-central1

# Reserve static IP
gcloud compute addresses create vpn-ip \
  --region=us-central1

# Create forwarding rules
gcloud compute forwarding-rules create vpn-rule-esp \
  --region=us-central1 \
  --ip-protocol=ESP \
  --address=vpn-ip \
  --target-vpn-gateway=my-vpn-gateway

# Create VPN tunnel
gcloud compute vpn-tunnels create tunnel-to-on-prem \
  --peer-address=203.0.113.1 \
  --shared-secret=mysecret \
  --ike-version=2 \
  --target-vpn-gateway=my-vpn-gateway \
  --region=us-central1

# Create route
gcloud compute routes create route-to-on-prem \
  --network=my-vpc \
  --next-hop-vpn-tunnel=tunnel-to-on-prem \
  --next-hop-vpn-tunnel-region=us-central1 \
  --destination-range=192.168.1.0/24
```

### VPC Peering

**GCP:**
```bash
# Create VPC peering from vpc-a to vpc-b
gcloud compute networks peerings create peer-a-to-b \
  --network=vpc-a \
  --peer-network=vpc-b \
  --peer-project=other-project-id

# Create reverse peering
gcloud compute networks peerings create peer-b-to-a \
  --network=vpc-b \
  --peer-network=vpc-a \
  --peer-project=my-project-id

# List peerings
gcloud compute networks peerings list
```

---

## Container & Kubernetes Services

### Google Kubernetes Engine (GKE)

**GCP's managed Kubernetes service - most feature-rich K8s offering**

**GCP:**
```bash
# Create GKE cluster (Standard mode)
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-medium \
  --disk-size=50 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5 \
  --enable-autorepair \
  --enable-autoupgrade \
  --enable-ip-alias \
  --network=my-vpc \
  --subnetwork=my-gke-subnet \
  --cluster-secondary-range-name=pods \
  --services-secondary-range-name=services

# Create regional cluster (HA control plane)
gcloud container clusters create my-regional-cluster \
  --region=us-central1 \
  --num-nodes=1 \
  --machine-type=e2-medium \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Create GKE Autopilot cluster (fully managed)
gcloud container clusters create-auto my-autopilot-cluster \
  --region=us-central1

# Get credentials for kubectl
gcloud container clusters get-credentials my-cluster --zone=us-central1-a

# Verify connection
kubectl get nodes

# Create node pool
gcloud container node-pools create high-mem-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=n2-highmem-4 \
  --num-nodes=2 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5

# Add GPU node pool
gcloud container node-pools create gpu-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --num-nodes=1

# Resize cluster
gcloud container clusters resize my-cluster \
  --num-nodes=5 \
  --zone=us-central1-a

# Upgrade cluster
gcloud container clusters upgrade my-cluster \
  --zone=us-central1-a \
  --master \
  --cluster-version=1.27.3-gke.100

# Delete cluster
gcloud container clusters delete my-cluster --zone=us-central1-a
```

**Deploy an application:**
```bash
# Create deployment
kubectl create deployment nginx --image=nginx:latest

# Expose deployment
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Get services
kubectl get services

# Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: gcr.io/my-project/web-app:v1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
EOF
```

**GKE Features Not in AWS EKS:**
- **Autopilot mode** - Fully managed, serverless K8s
- **Workload Identity** - Native GCP IAM integration
- **Binary Authorization** - Deploy-time security enforcement
- **Config Connector** - Manage GCP resources via K8s

**AWS Equivalent:**
```bash
# Create EKS cluster
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 5
```

### Cloud Run (Fully Managed Containers)

**GCP's unique serverless container platform** - run containers without managing infrastructure

**GCP:**
```bash
# Deploy container from Artifact Registry
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-app:latest \
  --platform=managed \
  --region=us-central1 \
  --allow-unauthenticated \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=100 \
  --concurrency=80 \
  --timeout=300s \
  --set-env-vars="ENV=production,DB_HOST=10.0.0.1"

# Deploy from source (buildpacks)
gcloud run deploy my-service \
  --source=. \
  --region=us-central1

# Update service
gcloud run services update my-service \
  --region=us-central1 \
  --memory=1Gi \
  --max-instances=200

# Set IAM permissions
gcloud run services add-iam-policy-binding my-service \
  --region=us-central1 \
  --member="allUsers" \
  --role="roles/run.invoker"

# Get service URL
gcloud run services describe my-service \
  --region=us-central1 \
  --format="value(status.url)"

# Deploy with custom domain
gcloud run domain-mappings create \
  --service=my-service \
  --domain=api.example.com \
  --region=us-central1

# Deploy with VPC connector (access VPC resources)
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-app:latest \
  --region=us-central1 \
  --vpc-connector=my-connector

# Set traffic split (blue/green deployment)
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-v2=50,my-service-v1=50

# List services
gcloud run services list

# View logs
gcloud run logs read my-service --region=us-central1
```

**Example Dockerfile for Cloud Run:**
```dockerfile
FROM node:18-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
ENV PORT=8080
EXPOSE 8080
CMD ["node", "server.js"]
```

**AWS Equivalent:** AWS App Runner or ECS Fargate
```bash
# AWS App Runner
aws apprunner create-service \
  --service-name my-service \
  --source-configuration file://source-config.json
```

### Artifact Registry / Container Registry

**GCP:**
```bash
# Create Artifact Registry repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker repository"

# Configure Docker auth
gcloud auth configure-docker us-central1-docker.pkg.dev

# Tag and push image
docker tag my-app:latest us-central1-docker.pkg.dev/my-project/my-repo/my-app:latest
docker push us-central1-docker.pkg.dev/my-project/my-repo/my-app:latest

# List images
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo

# For Container Registry (older, still supported):
docker tag my-app gcr.io/my-project/my-app:latest
docker push gcr.io/my-project/my-app:latest
```

**AWS Equivalent:**
```bash
# ECR
aws ecr create-repository --repository-name my-repo

# Login
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Push
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
```

---

## Serverless & Functions

### Cloud Functions

**GCP:**
```bash
# Deploy function (Gen 2 - recommended)
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --allow-unauthenticated \
  --memory=256MB \
  --timeout=60s \
  --max-instances=100 \
  --set-env-vars="DB_HOST=10.0.0.1,ENV=prod"

# Deploy with Pub/Sub trigger
gcloud functions deploy process-messages \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_pubsub \
  --trigger-topic=my-topic

# Deploy with Cloud Storage trigger
gcloud functions deploy process-upload \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_file \
  --trigger-bucket=my-bucket

# Deploy with Firestore trigger
gcloud functions deploy on-user-create \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=on_create \
  --trigger-event-filters="type=google.cloud.firestore.document.v1.created" \
  --trigger-event-filters="database=(default)" \
  --trigger-event-filters-path-pattern="document=users/{userId}"

# List functions
gcloud functions list

# Call function
gcloud functions call my-function \
  --region=us-central1 \
  --data='{"name":"World"}'

# View logs
gcloud functions logs read my-function --region=us-central1

# Delete function
gcloud functions delete my-function --region=us-central1
```

**Example Python function:**
```python
# main.py
import functions_framework
from flask import jsonify

@functions_framework.http
def main(request):
    """HTTP Cloud Function."""
    request_json = request.get_json(silent=True)
    name = request_json.get('name', 'World') if request_json else 'World'

    return jsonify({
        'message': f'Hello {name}!',
        'status': 'success'
    })

@functions_framework.cloud_event
def process_pubsub(cloud_event):
    """Pub/Sub triggered function."""
    import base64
    message = base64.b64decode(cloud_event.data["message"]["data"]).decode()
    print(f"Received message: {message}")

@functions_framework.cloud_event
def process_file(cloud_event):
    """Cloud Storage triggered function."""
    file_name = cloud_event.data["name"]
    bucket = cloud_event.data["bucket"]
    print(f"File {file_name} uploaded to {bucket}")
```

**requirements.txt:**
```
functions-framework==3.*
flask==3.*
google-cloud-storage==2.*
```

**AWS Equivalent:**
```bash
# Deploy Lambda function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.11 \
  --role arn:aws:iam::123456789:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### Cloud Tasks (SQS)

**GCP:**
```bash
# Create queue
gcloud tasks queues create my-queue \
  --location=us-central1 \
  --max-concurrent-dispatches=100 \
  --max-attempts=5

# Create HTTP task
gcloud tasks create-http-task \
  --queue=my-queue \
  --location=us-central1 \
  --url="https://example.com/process" \
  --method=POST \
  --header="Content-Type: application/json" \
  --body-content='{"data":"value"}' \
  --schedule-time="2024-01-01T12:00:00Z"

# Using Python SDK:
```python
from google.cloud import tasks_v2

client = tasks_v2.CloudTasksClient()
parent = client.queue_path('my-project', 'us-central1', 'my-queue')

task = {
    'http_request': {
        'http_method': tasks_v2.HttpMethod.POST,
        'url': 'https://example.com/process',
        'headers': {'Content-Type': 'application/json'},
        'body': '{"key":"value"}'.encode()
    }
}

response = client.create_task(request={'parent': parent, 'task': task})
print(f'Created task {response.name}')
```

**AWS Equivalent:**
```bash
# Create SQS queue
aws sqs create-queue --queue-name my-queue

# Send message
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/my-queue \
  --message-body "Hello"
```

### Cloud Pub/Sub (SNS/SQS)

**GCP:**
```bash
# Create topic
gcloud pubsub topics create my-topic

# Create subscription
gcloud pubsub subscriptions create my-subscription \
  --topic=my-topic \
  --ack-deadline=60 \
  --message-retention-duration=7d \
  --expiration-period=never

# Create push subscription
gcloud pubsub subscriptions create my-push-subscription \
  --topic=my-topic \
  --push-endpoint=https://example.com/webhook

# Publish message
gcloud pubsub topics publish my-topic \
  --message="Hello World" \
  --attribute=key1=value1,key2=value2

# Pull messages
gcloud pubsub subscriptions pull my-subscription \
  --auto-ack \
  --limit=10

# Using Python:
```python
from google.cloud import pubsub_v1
import json

# Publisher
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('my-project', 'my-topic')

data = json.dumps({'key': 'value'}).encode('utf-8')
future = publisher.publish(topic_path, data, attribute1='value1')
message_id = future.result()
print(f'Published message ID: {message_id}')

# Subscriber
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path('my-project', 'my-subscription')

def callback(message):
    print(f'Received message: {message.data.decode()}')
    print(f'Attributes: {message.attributes}')
    message.ack()

streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
print(f'Listening for messages on {subscription_path}...')

try:
    streaming_pull_future.result()
except KeyboardInterrupt:
    streaming_pull_future.cancel()
```

**Create dead-letter queue:**
```bash
# Create dead-letter topic
gcloud pubsub topics create my-topic-dlq

# Create dead-letter subscription
gcloud pubsub subscriptions create my-dlq-subscription \
  --topic=my-topic-dlq

# Update main subscription with DLQ
gcloud pubsub subscriptions update my-subscription \
  --dead-letter-topic=my-topic-dlq \
  --max-delivery-attempts=5
```

**AWS Equivalent:**
```bash
# SNS
aws sns create-topic --name my-topic
aws sns publish --topic-arn arn:aws:sns:us-east-1:123:my-topic --message "Hello"

# SQS
aws sqs create-queue --queue-name my-queue
aws sqs send-message --queue-url https://... --message-body "Hello"
```

---

## Big Data & Analytics

### BigQuery (Data Warehouse)

**GCP's serverless, highly scalable data warehouse**

**GCP:**
```bash
# Create dataset
bq mk --dataset \
  --location=US \
  --description="My dataset" \
  my_project:my_dataset

# Create table
bq mk --table \
  my_project:my_dataset.my_table \
  schema.json

# Schema file (schema.json):
# [
#   {"name": "name", "type": "STRING", "mode": "REQUIRED"},
#   {"name": "age", "type": "INTEGER", "mode": "NULLABLE"},
#   {"name": "email", "type": "STRING", "mode": "NULLABLE"}
# ]

# Load data from CSV
bq load \
  --source_format=CSV \
  --skip_leading_rows=1 \
  my_dataset.my_table \
  gs://my-bucket/data.csv \
  name:STRING,age:INTEGER,email:STRING

# Load from JSON
bq load \
  --source_format=NEWLINE_DELIMITED_JSON \
  my_dataset.my_table \
  gs://my-bucket/data.json

# Query data
bq query --use_legacy_sql=false '
  SELECT name, COUNT(*) as count
  FROM `my_project.my_dataset.my_table`
  GROUP BY name
  ORDER BY count DESC
  LIMIT 10
'

# Export query results
bq query --use_legacy_sql=false \
  --destination_table=my_dataset.results \
  --replace \
  'SELECT * FROM `my_dataset.my_table` WHERE age > 25'

# Extract to GCS
bq extract \
  --destination_format=CSV \
  my_dataset.results \
  gs://my-bucket/results/*.csv
```

**Using Python:**
```python
from google.cloud import bigquery

client = bigquery.Client()

# Create table
schema = [
    bigquery.SchemaField("name", "STRING", mode="REQUIRED"),
    bigquery.SchemaField("age", "INTEGER", mode="NULLABLE"),
    bigquery.SchemaField("email", "STRING", mode="NULLABLE"),
]

table_id = "my_project.my_dataset.my_table"
table = bigquery.Table(table_id, schema=schema)
table = client.create_table(table)

# Insert rows
rows_to_insert = [
    {"name": "John", "age": 30, "email": "john@example.com"},
    {"name": "Jane", "age": 25, "email": "jane@example.com"},
]
errors = client.insert_rows_json(table_id, rows_to_insert)

# Query
query = """
    SELECT name, age
    FROM `my_project.my_dataset.my_table`
    WHERE age > 25
    ORDER BY age DESC
"""
query_job = client.query(query)
results = query_job.result()

for row in results:
    print(f"{row.name}: {row.age}")

# Query with parameters
query = """
    SELECT name, age
    FROM `my_project.my_dataset.my_table`
    WHERE age > @min_age
"""
job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ScalarQueryParameter("min_age", "INT64", 25)
    ]
)
query_job = client.query(query, job_config=job_config)

# Load from GCS
job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.CSV,
    skip_leading_rows=1,
    autodetect=True,
)
uri = "gs://my-bucket/data.csv"
load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)
load_job.result()

# Create view
view_id = "my_project.my_dataset.my_view"
view = bigquery.Table(view_id)
view.view_query = "SELECT name, age FROM `my_project.my_dataset.my_table` WHERE age > 25"
view = client.create_table(view)
```

**Advanced BigQuery Features:**
```sql
-- Partitioned table (by date)
CREATE TABLE my_dataset.events (
  event_date DATE,
  user_id STRING,
  event_type STRING
)
PARTITION BY event_date;

-- Clustered table
CREATE TABLE my_dataset.events (
  event_date DATE,
  user_id STRING,
  event_type STRING
)
PARTITION BY event_date
CLUSTER BY user_id, event_type;

-- Streaming inserts (real-time)
-- Use Python client with streaming buffer

-- Query external data (Federated queries)
CREATE EXTERNAL TABLE my_dataset.external_table
OPTIONS (
  format = 'CSV',
  uris = ['gs://my-bucket/data*.csv'],
  skip_leading_rows = 1
);

-- Machine Learning in BigQuery
CREATE OR REPLACE MODEL my_dataset.linear_model
OPTIONS(model_type='linear_reg', input_label_cols=['label']) AS
SELECT * FROM my_dataset.training_data;

-- Predict
SELECT * FROM ML.PREDICT(MODEL my_dataset.linear_model,
  (SELECT * FROM my_dataset.test_data));
```

**AWS Equivalent:** Redshift or Athena
```bash
# Athena query
aws athena start-query-execution \
  --query-string "SELECT * FROM my_table LIMIT 10" \
  --result-configuration OutputLocation=s3://my-bucket/results/
```

### Dataflow (Apache Beam)

**GCP's fully managed stream and batch data processing**

**Python Beam example:**
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

# Define pipeline options
options = PipelineOptions(
    project='my-project',
    runner='DataflowRunner',
    region='us-central1',
    temp_location='gs://my-bucket/temp',
    staging_location='gs://my-bucket/staging'
)

# Create pipeline
with beam.Pipeline(options=options) as p:
    (p
     | 'Read from GCS' >> beam.io.ReadFromText('gs://my-bucket/input/*.txt')
     | 'Transform' >> beam.Map(lambda x: x.upper())
     | 'Filter' >> beam.Filter(lambda x: len(x) > 0)
     | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
         'my_project:my_dataset.my_table',
         schema='word:STRING,count:INTEGER',
         write_disposition=beam.io.BigQueryDisposition.WRITE_TRUNCATE
     ))

# Word count example
with beam.Pipeline(options=options) as p:
    (p
     | 'Read' >> beam.io.ReadFromText('gs://my-bucket/input.txt')
     | 'Split' >> beam.FlatMap(lambda x: x.split())
     | 'PairWithOne' >> beam.Map(lambda x: (x, 1))
     | 'GroupAndSum' >> beam.CombinePerKey(sum)
     | 'Write' >> beam.io.WriteToText('gs://my-bucket/output'))
```

**Streaming pipeline (Pub/Sub to BigQuery):**
```python
with beam.Pipeline(options=options) as p:
    (p
     | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(
         subscription='projects/my-project/subscriptions/my-sub')
     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))
     | 'Parse JSON' >> beam.Map(json.loads)
     | 'Add timestamp' >> beam.Map(lambda x: {**x, 'processed_at': str(datetime.now())})
     | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
         'my_project:my_dataset.streaming_table',
         schema=table_schema,
         write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
     ))
```

**AWS Equivalent:** Kinesis Data Analytics, AWS Glue, or EMR

### Dataproc (Managed Hadoop/Spark)

**GCP:**
```bash
# Create Dataproc cluster
gcloud dataproc clusters create my-cluster \
  --region=us-central1 \
  --zone=us-central1-a \
  --master-machine-type=n1-standard-4 \
  --master-boot-disk-size=500 \
  --num-workers=2 \
  --worker-machine-type=n1-standard-4 \
  --worker-boot-disk-size=500 \
  --image-version=2.1-debian11 \
  --enable-component-gateway \
  --optional-components=JUPYTER

# Submit PySpark job
gcloud dataproc jobs submit pyspark \
  --cluster=my-cluster \
  --region=us-central1 \
  gs://my-bucket/spark-job.py

# Submit with arguments
gcloud dataproc jobs submit pyspark \
  --cluster=my-cluster \
  --region=us-central1 \
  gs://my-bucket/spark-job.py \
  -- arg1 arg2

# Delete cluster
gcloud dataproc clusters delete my-cluster --region=us-central1
```

**Example PySpark job:**
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("MyApp").getOrCreate()

# Read from GCS
df = spark.read.csv("gs://my-bucket/data.csv", header=True, inferSchema=True)

# Transform
result = df.groupBy("category").count()

# Write to BigQuery
result.write \
    .format("bigquery") \
    .option("table", "my_project:my_dataset.results") \
    .mode("overwrite") \
    .save()
```

**AWS Equivalent:** EMR
```bash
# Create EMR cluster
aws emr create-cluster \
  --name "My cluster" \
  --release-label emr-6.10.0 \
  --applications Name=Spark \
  --instance-type m5.xlarge \
  --instance-count 3
```

---

## Machine Learning & AI

### Vertex AI (ML Platform)

**GCP's unified ML platform**

**Train custom model:**
```python
from google.cloud import aiplatform

aiplatform.init(project='my-project', location='us-central1')

# Create custom training job
job = aiplatform.CustomTrainingJob(
    display_name='my-training-job',
    script_path='train.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-cpu.2-12:latest',
    requirements=['pandas', 'scikit-learn'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/tf2-cpu.2-12:latest'
)

model = job.run(
    dataset=my_dataset,
    model_display_name='my-model',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    replica_count=1,
    machine_type='n1-standard-4'
)

# Deploy model
endpoint = model.deploy(
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5
)

# Make prediction
prediction = endpoint.predict(instances=[...])
```

**AutoML:**
```python
# AutoML Tables
dataset = aiplatform.TabularDataset.create(
    display_name='my-dataset',
    gcs_source='gs://my-bucket/data.csv'
)

job = aiplatform.AutoMLTabularTrainingJob(
    display_name='my-automl-job',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-prc'
)

model = job.run(
    dataset=dataset,
    target_column='label',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=1000
)
```

**Pre-trained APIs:**
```python
# Vision API
from google.cloud import vision

client = vision.ImageAnnotatorClient()

with open('image.jpg', 'rb') as image_file:
    content = image_file.read()

image = vision.Image(content=content)
response = client.label_detection(image=image)

for label in response.label_annotations:
    print(f'{label.description}: {label.score}')

# Natural Language API
from google.cloud import language_v1

client = language_v1.LanguageServiceClient()

text = "Google Cloud Platform is awesome!"
document = language_v1.Document(
    content=text,
    type_=language_v1.Document.Type.PLAIN_TEXT
)

sentiment = client.analyze_sentiment(request={'document': document}).document_sentiment
print(f'Sentiment score: {sentiment.score}, magnitude: {sentiment.magnitude}')

# Translation API
from google.cloud import translate_v2

client = translate_v2.Client()

text = "Hello, world!"
result = client.translate(text, target_language='es')
print(f"Translation: {result['translatedText']}")

# Speech-to-Text API
from google.cloud import speech

client = speech.SpeechClient()

with open('audio.wav', 'rb') as audio_file:
    content = audio_file.read()

audio = speech.RecognitionAudio(content=content)
config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=16000,
    language_code='en-US'
)

response = client.recognize(config=config, audio=audio)

for result in response.results:
    print(f'Transcript: {result.alternatives[0].transcript}')
```

**AWS Equivalent:** SageMaker, Rekognition, Comprehend, Translate, Transcribe

---

## Security & Compliance

### Cloud KMS (Key Management)

**GCP:**
```bash
# Create keyring
gcloud kms keyrings create my-keyring \
  --location=us-central1

# Create key
gcloud kms keys create my-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --purpose=encryption

# Encrypt data
echo "sensitive data" | gcloud kms encrypt \
  --location=us-central1 \
  --keyring=my-keyring \
  --key=my-key \
  --plaintext-file=- \
  --ciphertext-file=encrypted.bin

# Decrypt data
gcloud kms decrypt \
  --location=us-central1 \
  --keyring=my-keyring \
  --key=my-key \
  --ciphertext-file=encrypted.bin \
  --plaintext-file=-

# Grant encryption permission
gcloud kms keys add-iam-policy-binding my-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --member=serviceAccount:my-sa@my-project.iam.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter
```

**Using Python:**
```python
from google.cloud import kms

client = kms.KeyManagementServiceClient()

key_name = client.crypto_key_path('my-project', 'us-central1', 'my-keyring', 'my-key')

# Encrypt
plaintext = b"sensitive data"
response = client.encrypt(request={'name': key_name, 'plaintext': plaintext})
ciphertext = response.ciphertext

# Decrypt
response = client.decrypt(request={'name': key_name, 'ciphertext': ciphertext})
plaintext = response.plaintext
```

### Secret Manager

**GCP:**
```bash
# Create secret
echo -n "my-secret-value" | gcloud secrets create my-secret \
  --data-file=-

# Add version
echo -n "new-secret-value" | gcloud secrets versions add my-secret \
  --data-file=-

# Access secret
gcloud secrets versions access latest --secret=my-secret

# Grant access
gcloud secrets add-iam-policy-binding my-secret \
  --member=serviceAccount:my-sa@my-project.iam.gserviceaccount.com \
  --role=roles/secretmanager.secretAccessor

# List secrets
gcloud secrets list
```

**Using Python:**
```python
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
project_id = 'my-project'

# Create secret
parent = f"projects/{project_id}"
secret = client.create_secret(
    request={
        "parent": parent,
        "secret_id": "my-secret",
        "secret": {"replication": {"automatic": {}}},
    }
)

# Add secret version
parent = secret.name
payload = "my-secret-value".encode("UTF-8")
version = client.add_secret_version(
    request={"parent": parent, "payload": {"data": payload}}
)

# Access secret
name = f"projects/{project_id}/secrets/my-secret/versions/latest"
response = client.access_secret_version(request={"name": name})
secret_value = response.payload.data.decode("UTF-8")
```

**AWS Equivalent:** Secrets Manager, Systems Manager Parameter Store

### Cloud Armor (WAF)

**GCP:**
```bash
# Create security policy
gcloud compute security-policies create my-policy \
  --description="My security policy"

# Add rule to block specific IP
gcloud compute security-policies rules create 1000 \
  --security-policy=my-policy \
  --expression="origin.ip == '203.0.113.0'" \
  --action=deny-403

# Add rate limiting rule
gcloud compute security-policies rules create 2000 \
  --security-policy=my-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600

# Attach to backend service
gcloud compute backend-services update web-backend-service \
  --security-policy=my-policy \
  --global
```

**AWS Equivalent:** AWS WAF

### VPC Service Controls

**Create security perimeter around GCP resources:**

```bash
# Create access policy (via console or Terraform)
# Create service perimeter
gcloud access-context-manager perimeters create my-perimeter \
  --title="My Perimeter" \
  --resources=projects/123456789 \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com \
  --policy=accessPolicies/123456789
```

**AWS Equivalent:** VPC Endpoints, SCPs

---

## Monitoring & Logging

### Cloud Monitoring (CloudWatch)

**GCP:**
```bash
# List metrics
gcloud monitoring metrics-descriptors list

# Create uptime check
gcloud monitoring uptime create http-check \
  --display-name="My HTTP Check" \
  --resource-type=uptime-url \
  --host=example.com \
  --path=/health

# Create alert policy (via console or API)
```

**Using Python:**
```python
from google.cloud import monitoring_v3
import time

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/my-project"

# Write custom metric
series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/my_metric"
series.resource.type = "global"

now = time.time()
seconds = int(now)
nanos = int((now - seconds) * 10 ** 9)
interval = monitoring_v3.TimeInterval(
    {"end_time": {"seconds": seconds, "nanos": nanos}}
)
point = monitoring_v3.Point(
    {"interval": interval, "value": {"double_value": 123.45}}
)
series.points = [point]

client.create_time_series(name=project_name, time_series=[series])

# Read metrics
interval = monitoring_v3.TimeInterval(
    {
        "end_time": {"seconds": int(time.time())},
        "start_time": {"seconds": int(time.time() - 3600)},
    }
)
results = client.list_time_series(
    request={
        "name": project_name,
        "filter": 'metric.type = "compute.googleapis.com/instance/cpu/utilization"',
        "interval": interval,
        "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
    }
)
for result in results:
    print(result)
```

### Cloud Logging (CloudWatch Logs)

**GCP:**
```bash
# Read logs
gcloud logging read "resource.type=gce_instance" \
  --limit=10 \
  --format=json

# Read logs with filter
gcloud logging read \
  'resource.type="gce_instance" AND severity>=ERROR' \
  --limit=50

# Create log sink (export to BigQuery)
gcloud logging sinks create my-sink \
  bigquery.googleapis.com/projects/my-project/datasets/my_logs_dataset \
  --log-filter='resource.type="gce_instance"'

# Create sink to Cloud Storage
gcloud logging sinks create my-storage-sink \
  storage.googleapis.com/my-logs-bucket \
  --log-filter='severity>=WARNING'

# Write log entry
gcloud logging write my-log "Test message" --severity=INFO
```

**Using Python:**
```python
from google.cloud import logging
import json

# Setup logging client
logging_client = logging.Client()
logger = logging_client.logger('my-application')

# Write structured log
logger.log_struct(
    {
        "message": "User login",
        "user_id": "12345",
        "ip_address": "203.0.113.1"
    },
    severity='INFO'
)

# Write text log
logger.log_text("Simple text log", severity='WARNING')

# List logs
for entry in logging_client.list_entries(max_results=10):
    print(f'{entry.timestamp}: {entry.payload}')

# Query logs
FILTER = 'resource.type="gce_instance" AND severity>=ERROR'
for entry in logging_client.list_entries(filter_=FILTER):
    print(entry.payload)
```

### Cloud Trace (X-Ray)

**Distributed tracing:**

```python
from google.cloud import trace_v2
from google.cloud.trace_v2 import TraceServiceClient

client = TraceServiceClient()

# Create span
project_id = 'my-project'
trace_id = 'abc123'
span_id = 'span123'

span = {
    'name': f'projects/{project_id}/traces/{trace_id}/spans/{span_id}',
    'span_id': span_id,
    'display_name': {'value': 'my-span'},
    'start_time': {'seconds': int(time.time())},
    'end_time': {'seconds': int(time.time()) + 1}
}

client.create_span(name=span['name'], **span)
```

### Cloud Profiler

**Continuous profiling:**

```python
import googlecloudprofiler

try:
    googlecloudprofiler.start(
        service='my-service',
        service_version='1.0.0',
        verbose=3
    )
except Exception as exc:
    print(exc)
```

**AWS Equivalents:** CloudWatch, X-Ray, CloudWatch Logs Insights

---

## DevOps & CI/CD

### Cloud Build (CodeBuild)

**GCP:**
```bash
# Submit build
gcloud builds submit \
  --tag=gcr.io/my-project/my-app:latest \
  .

# Submit with custom config
gcloud builds submit \
  --config=cloudbuild.yaml \
  .

# Create trigger from GitHub
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml

# List builds
gcloud builds list

# View build logs
gcloud builds log <BUILD_ID>
```

**cloudbuild.yaml:**
```yaml
steps:
  # Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA', '.']

  # Run tests
  - name: 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA'
    args: ['npm', 'test']

  # Push image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA']

  # Deploy to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'my-service'
      - '--image=gcr.io/$PROJECT_ID/my-app:$SHORT_SHA'
      - '--region=us-central1'
      - '--platform=managed'

images:
  - 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA'

options:
  machineType: 'N1_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY

timeout: '1200s'
```

**Multi-stage build:**
```yaml
steps:
  # Install dependencies
  - name: 'node:18'
    entrypoint: npm
    args: ['install']

  # Run linter
  - name: 'node:18'
    entrypoint: npm
    args: ['run', 'lint']

  # Run unit tests
  - name: 'node:18'
    entrypoint: npm
    args: ['test']

  # Build production bundle
  - name: 'node:18'
    entrypoint: npm
    args: ['run', 'build']

  # Build container
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/app:$COMMIT_SHA', '.']

  # Push to registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/app:$COMMIT_SHA']

  # Deploy to GKE
  - name: 'gcr.io/cloud-builders/kubectl'
    args:
      - 'set'
      - 'image'
      - 'deployment/my-app'
      - 'my-app=gcr.io/$PROJECT_ID/app:$COMMIT_SHA'
    env:
      - 'CLOUDSDK_COMPUTE_REGION=us-central1'
      - 'CLOUDSDK_CONTAINER_CLUSTER=my-cluster'

substitutions:
  _ENV: 'production'

secrets:
  - kmsKeyName: projects/my-project/locations/global/keyRings/my-keyring/cryptoKeys/my-key
    secretEnv:
      API_KEY: CiQA...encrypted...==
```

### Cloud Deploy (Continuous Delivery)

**GCP's managed CD service:**

```yaml
# clouddeploy.yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-pipeline
serialPipeline:
  stages:
  - targetId: dev
    profiles: []
  - targetId: staging
    profiles: []
  - targetId: prod
    profiles: []
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: dev
deployParameters: {}
run:
  location: projects/my-project/locations/us-central1
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
requireApproval: true
run:
  location: projects/my-project/locations/us-central1
```

```bash
# Create delivery pipeline
gcloud deploy apply --file=clouddeploy.yaml --region=us-central1

# Create release
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-pipeline \
  --region=us-central1 \
  --images=my-app=gcr.io/my-project/my-app:v1

# Promote to next stage
gcloud deploy releases promote \
  --release=release-001 \
  --delivery-pipeline=my-pipeline \
  --region=us-central1
```

### Infrastructure as Code

**Deployment Manager:**
```yaml
# compute-engine.yaml
resources:
- name: my-vm
  type: compute.v1.instance
  properties:
    zone: us-central1-a
    machineType: zones/us-central1-a/machineTypes/e2-medium
    disks:
    - deviceName: boot
      type: PERSISTENT
      boot: true
      autoDelete: true
      initializeParams:
        sourceImage: projects/debian-cloud/global/images/family/debian-11
    networkInterfaces:
    - network: global/networks/default
      accessConfigs:
      - name: External NAT
        type: ONE_TO_ONE_NAT
```

```bash
# Deploy
gcloud deployment-manager deployments create my-deployment \
  --config=compute-engine.yaml

# Update
gcloud deployment-manager deployments update my-deployment \
  --config=compute-engine.yaml

# Delete
gcloud deployment-manager deployments delete my-deployment
```

**Terraform (Recommended):**
```hcl
# main.tf
provider "google" {
  project = "my-project"
  region  = "us-central1"
}

resource "google_compute_instance" "vm" {
  name         = "my-vm"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
  EOF
}

resource "google_storage_bucket" "bucket" {
  name     = "my-unique-bucket-name"
  location = "US"

  lifecycle_rule {
    action {
      type = "Delete"
    }
    condition {
      age = 30
    }
  }
}

resource "google_sql_database_instance" "mysql" {
  name             = "my-mysql-instance"
  database_version = "MYSQL_8_0"
  region           = "us-central1"

  settings {
    tier = "db-f1-micro"

    backup_configuration {
      enabled = true
      start_time = "03:00"
    }
  }
}
```

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

**AWS Equivalent:** CloudFormation, AWS CDK

---

## Cost Management

### Pricing Concepts

**Key Differences from AWS:**
- **Sustained use discounts** - Automatic discounts for running instances >25% of month
- **Committed use discounts** - 1 or 3-year commitments (like Reserved Instances)
- **Per-second billing** - Most services bill per second (AWS often per hour)
- **No data transfer charges within same region** (different zones OK)

### Cost Management Tools

```bash
# Set budget alert
gcloud billing budgets create \
  --billing-account=0X0X0X-0X0X0X-0X0X0X \
  --display-name="Monthly Budget" \
  --budget-amount=1000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=90 \
  --threshold-rule=percent=100

# Export billing to BigQuery (via console)
# Then query:
bq query --use_legacy_sql=false '
  SELECT
    service.description,
    SUM(cost) as total_cost
  FROM `my-project.billing_export.gcp_billing_export_v1_XXXXX`
  WHERE DATE(_PARTITIONTIME) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
  GROUP BY service.description
  ORDER BY total_cost DESC
  LIMIT 10
'

# Use Recommender API for cost optimization
gcloud recommender recommendations list \
  --project=my-project \
  --location=us-central1 \
  --recommender=google.compute.instance.MachineTypeRecommender
```

### Cost Optimization Tips

1. **Use committed use discounts** for steady workloads (57% discount)
2. **Use preemptible/spot VMs** for fault-tolerant workloads (91% discount)
3. **Right-size instances** using Recommender API
4. **Use Cloud Storage lifecycle policies** to move to cheaper storage classes
5. **Delete unused resources**:
   ```bash
   # Find unused disks
   gcloud compute disks list --filter="-users:*"

   # Find unused IPs
   gcloud compute addresses list --filter="status:RESERVED"
   ```
6. **Use labels** for cost attribution:
   ```bash
   gcloud compute instances create my-vm \
     --labels=env=prod,team=backend,cost-center=engineering
   ```

---

## Best Practices & Patterns

### Project Organization

**Recommended structure:**
```
Organization
├── Shared Services Folder
│   ├── Networking Project (VPC, Cloud DNS)
│   ├── Security Project (KMS, Secret Manager)
│   └── Monitoring Project (centralized logging)
├── Production Folder
│   ├── Prod Frontend Project
│   ├── Prod Backend Project
│   └── Prod Data Project
├── Staging Folder
│   └── Staging Project
└── Development Folder
    ├── Dev Team A Project
    └── Dev Team B Project
```

### Security Best Practices

1. **Use service accounts** for applications, not user accounts
2. **Enable VPC Service Controls** for sensitive data
3. **Use Secret Manager** for credentials, not environment variables
4. **Enable Cloud Audit Logs** for compliance
5. **Use Organization Policy** to enforce constraints:
   ```bash
   # Restrict VM external IPs
   gcloud resource-manager org-policies set-policy \
     --organization=123456789 \
     policy.yaml
   ```
6. **Implement least privilege IAM**
7. **Use customer-managed encryption keys (CMEK)** for sensitive data
8. **Enable Binary Authorization** for container deployment

### Networking Best Practices

1. **Use Shared VPC** for centralized network management
2. **Implement VPC firewall rules hierarchy**:
   - Deny-all rule (priority 65534)
   - Specific allow rules (priority 1000-9999)
   - Emergency deny rules (priority 0-999)
3. **Use Cloud NAT** for outbound traffic from private instances
4. **Implement Private Google Access** for VMs without external IPs
5. **Use VPC Peering** or VPN for multi-VPC connectivity

### High Availability Patterns

1. **Use regional resources** (regional Compute instance groups, regional Cloud SQL)
2. **Implement health checks** for load balancers
3. **Use multi-region** for critical data (Cloud Storage, Cloud Spanner)
4. **Implement auto-healing** for instance groups
5. **Use GKE regional clusters** for K8s workloads
6. **Implement backup strategies**:
   ```bash
   # Automated snapshots
   gcloud compute resource-policies create snapshot-schedule daily-snapshot \
     --region=us-central1 \
     --max-retention-days=7 \
     --on-source-disk-delete=keep-auto-snapshots \
     --daily-schedule \
     --start-time=03:00
   ```

### Migration from AWS

**Common migration paths:**

1. **Lift and Shift**: EC2 → Compute Engine
2. **Re-platform**: RDS → Cloud SQL, S3 → Cloud Storage
3. **Re-architect**: Lambda → Cloud Functions/Cloud Run
4. **Modernize**: ECS → GKE, traditional apps → Cloud Run

**Tools:**
- **Migrate for Compute Engine** (formerly Velostrata) - VM migration
- **BigQuery Data Transfer Service** - migrate from Redshift, Teradata
- **Database Migration Service** - migrate from AWS RDS
- **Transfer Service** - migrate data from S3 to GCS

**Example S3 to GCS migration:**
```bash
# Using Transfer Service (via console or API)
# Or use gsutil:
gsutil -m rsync -r s3://my-aws-bucket gs://my-gcs-bucket
```

### Useful gcloud Commands

```bash
# Show current configuration
gcloud config list

# Switch projects
gcloud config set project my-other-project

# Multiple configurations
gcloud config configurations create dev
gcloud config configurations activate dev
gcloud config set project dev-project

# List all resources in project
gcloud asset search-all-resources --scope=projects/my-project

# Estimate costs (using Cloud Billing Catalog API)
gcloud billing accounts list

# Find quotas
gcloud compute project-info describe --project=my-project

# Increase quota (request via console)

# Filter and format output
gcloud compute instances list \
  --filter="zone:us-central1-a AND status:RUNNING" \
  --format="table(name,machineType,status)"

# Use --dry-run to preview changes
gcloud compute instances delete my-vm --dry-run

# Output as JSON
gcloud compute instances list --format=json

# Chain commands
gcloud compute instances list --format="value(name)" | \
  xargs -I {} gcloud compute instances delete {} --zone=us-central1-a --quiet
```

---

## Quick Reference: AWS to GCP Service Mapping

| Category | AWS | GCP | Notes |
|----------|-----|-----|-------|
| **Compute** |
| Virtual Machines | EC2 | Compute Engine | Similar concepts |
| Serverless Containers | Fargate | Cloud Run | GCP more flexible |
| Kubernetes | EKS | GKE | GCP more features |
| Functions | Lambda | Cloud Functions | Similar, GCP has Cloud Run for containers |
| **Storage** |
| Object Storage | S3 | Cloud Storage | Very similar |
| Block Storage | EBS | Persistent Disks | Similar |
| File Storage | EFS | Filestore | Similar NFS |
| Archive | Glacier | Cloud Storage Archive | GCP simpler |
| **Database** |
| Relational | RDS | Cloud SQL | Similar |
| NoSQL Document | DynamoDB | Firestore | Different data model |
| NoSQL Wide-Column | Cassandra on AWS | Bigtable | Similar to HBase |
| Data Warehouse | Redshift | BigQuery | GCP serverless |
| In-Memory | ElastiCache | Memorystore | Similar |
| **Networking** |
| VPC | VPC | VPC | GCP global, AWS regional |
| Load Balancer | ELB/ALB/NLB | Cloud Load Balancing | GCP global L7 LB |
| CDN | CloudFront | Cloud CDN | Similar |
| DNS | Route 53 | Cloud DNS | Similar |
| VPN | VPN Gateway | Cloud VPN | Similar |
| **Big Data** |
| Stream Processing | Kinesis | Dataflow | GCP uses Beam |
| Hadoop/Spark | EMR | Dataproc | Similar |
| Data Warehouse | Redshift | BigQuery | GCP serverless |
| **ML/AI** |
| ML Platform | SageMaker | Vertex AI | GCP more integrated |
| Vision | Rekognition | Vision API | Similar |
| Language | Comprehend | Natural Language API | Similar |
| **DevOps** |
| CI/CD | CodePipeline | Cloud Build | Similar |
| Container Registry | ECR | Artifact Registry | GCP newer, more features |
| **Security** |
| IAM | IAM | Cloud IAM | Different model |
| Secrets | Secrets Manager | Secret Manager | Similar |
| Encryption | KMS | Cloud KMS | Similar |
| **Monitoring** |
| Monitoring | CloudWatch | Cloud Monitoring | Similar |
| Logging | CloudWatch Logs | Cloud Logging | Similar |
| Tracing | X-Ray | Cloud Trace | Similar |

---

## Learning Path & Certifications

### Recommended Learning Path

1. **Fundamentals** (Week 1-2)
   - GCP Console navigation
   - gcloud CLI basics
   - Project and billing setup
   - IAM and service accounts

2. **Core Services** (Week 3-4)
   - Compute Engine
   - Cloud Storage
   - Cloud SQL
   - VPC networking

3. **Advanced Compute** (Week 5-6)
   - GKE
   - Cloud Run
   - Cloud Functions

4. **Data Services** (Week 7-8)
   - BigQuery
   - Pub/Sub
   - Dataflow

5. **Production Practices** (Week 9-10)
   - Monitoring and logging
   - Security best practices
   - Cost optimization
   - CI/CD with Cloud Build

### GCP Certifications

1. **Associate Cloud Engineer** - Entry level
2. **Professional Cloud Architect** - Design and architecture
3. **Professional Data Engineer** - Data pipelines and ML
4. **Professional Cloud Developer** - Application development
5. **Professional Cloud DevOps Engineer** - SRE and DevOps
6. **Professional Cloud Security Engineer** - Security specialist

### Resources

- **Official Documentation**: cloud.google.com/docs
- **Qwiklabs**: hands-on labs
- **Coursera**: Google Cloud training courses
- **Cloud Skills Boost**: certification prep
- **YouTube**: Google Cloud Tech channel

---

## Conclusion

This guide covers the essential GCP services and their AWS equivalents. Key takeaways:

1. **GCP's strengths**: BigQuery, GKE, Cloud Run, data analytics, ML/AI
2. **AWS equivalents exist** for most GCP services, but implementation differs
3. **Global vs Regional**: GCP VPCs are global; AWS VPCs are regional
4. **Pricing**: GCP offers per-second billing and automatic sustained-use discounts
5. **Kubernetes**: GKE is the most feature-rich managed K8s service
6. **Data & Analytics**: BigQuery and Dataflow are GCP differentiators

Start with core services (Compute, Storage, Networking) and expand to specialized services as needed. Good luck on your GCP journey!
