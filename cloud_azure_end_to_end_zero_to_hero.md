# Azure End-to-End Zero to Hero Guide
## For AWS Experts Transitioning to Azure

---

## Table of Contents
1. [Azure Fundamentals & Core Concepts](#1-azure-fundamentals--core-concepts)
2. [Identity & Access Management](#2-identity--access-management)
3. [Compute Services](#3-compute-services)
4. [Storage Services](#4-storage-services)
5. [Networking](#5-networking)
6. [Databases](#6-databases)
7. [Containers & Orchestration](#7-containers--orchestration)
8. [Serverless & Event-Driven Architecture](#8-serverless--event-driven-architecture)
9. [Monitoring & Management](#9-monitoring--management)
10. [DevOps & CI/CD](#10-devops--cicd)
11. [Security & Compliance](#11-security--compliance)
12. [Cost Management](#12-cost-management)
13. [Advanced Architectures](#13-advanced-architectures)

---

## 1. Azure Fundamentals & Core Concepts

### 1.1 Azure Hierarchy vs AWS Hierarchy

**Azure Organization Structure:**
```
Management Groups
    └── Subscriptions (~ AWS Accounts)
        └── Resource Groups (~ AWS Tags/Logical Grouping)
            └── Resources (VMs, DBs, etc.)
```

**AWS Equivalent:**
```
AWS Organization
    └── Organizational Units (OUs)
        └── AWS Accounts
            └── Resources (tagged/grouped)
```

**Key Difference:** Azure has **Resource Groups** - mandatory logical containers for resources. Every resource MUST belong to a resource group.

### 1.2 Azure Regions & Availability

| Azure Concept | AWS Equivalent | Description |
|---------------|----------------|-------------|
| **Region** | Region | Geographic area with multiple datacenters |
| **Availability Zone** | Availability Zone | Physically separate datacenters within a region |
| **Region Pair** | *N/A* | Two regions paired for disaster recovery (800+ miles apart) |
| **Geo** | *N/A* | Geographic area containing multiple regions |

**Example: Creating a Resource Group**

```bash
# Azure CLI
az group create \
  --name myResourceGroup \
  --location eastus

# AWS CLI Equivalent (no direct equivalent - using tags)
aws resourcegroupstaggingapi create-tags \
  --resource-arn <arn> \
  --tags Environment=Production
```

### 1.3 Azure Resource Manager (ARM) vs AWS CloudFormation

**Azure ARM Template:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "mystorageaccount",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

**AWS CloudFormation Equivalent:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: my-s3-bucket
```

**Modern Alternative - Bicep (Azure) vs Terraform:**
```bicep
// Azure Bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: 'mystorageaccount'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

---

## 2. Identity & Access Management

### 2.1 Azure AD (Entra ID) vs AWS IAM

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure AD (Entra ID)** | IAM + AWS SSO + Cognito | Identity provider & directory service |
| **Service Principal** | IAM Role | Identity for applications |
| **Managed Identity** | IAM Role (for EC2/Lambda) | Automatic identity for Azure resources |
| **Azure RBAC** | IAM Policies | Authorization & permissions |
| **Azure AD Groups** | IAM Groups | Group-based permissions |

### 2.2 Role-Based Access Control (RBAC)

**Built-in Roles:**
- **Owner** = AWS AdministratorAccess
- **Contributor** = AWS PowerUserAccess (can't manage permissions)
- **Reader** = AWS ReadOnlyAccess

**Example: Assign Role to User**

```bash
# Azure CLI
az role assignment create \
  --assignee user@example.com \
  --role "Contributor" \
  --scope "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup"

# AWS CLI Equivalent
aws iam attach-user-policy \
  --user-name username \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

### 2.3 Managed Identities (System vs User-Assigned)

**System-Assigned Identity:**
```bash
# Enable on a VM
az vm identity assign \
  --name myVM \
  --resource-group myResourceGroup

# AWS Equivalent: Attach IAM Role to EC2
aws ec2 associate-iam-instance-profile \
  --instance-id i-1234567890abcdef0 \
  --iam-instance-profile Name=MyRole
```

**User-Assigned Identity:**
```bash
# Create managed identity (can be shared across resources)
az identity create \
  --name myManagedIdentity \
  --resource-group myResourceGroup

# Assign to multiple VMs
az vm identity assign \
  --name myVM1 \
  --resource-group myResourceGroup \
  --identities myManagedIdentity
```

**Access Azure Resources from Code:**

```python
# Azure SDK with Managed Identity
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

credential = DefaultAzureCredential()
blob_service = BlobServiceClient(
    account_url="https://mystorageaccount.blob.core.windows.net",
    credential=credential
)

# AWS SDK Equivalent (boto3)
import boto3
s3 = boto3.client('s3')  # Automatically uses IAM role
```

---

## 3. Compute Services

### 3.1 Service Comparison

| Azure | AWS | Use Case |
|-------|-----|----------|
| **Virtual Machines (VMs)** | EC2 | IaaS compute |
| **VM Scale Sets (VMSS)** | Auto Scaling Groups | Auto-scaling VMs |
| **Azure App Service** | Elastic Beanstalk | PaaS web apps |
| **Azure Functions** | Lambda | Serverless functions |
| **Azure Container Instances (ACI)** | Fargate (without ECS) | Serverless containers |
| **Azure Batch** | AWS Batch | Batch processing |

### 3.2 Virtual Machines

**VM Sizes (vs EC2 Instance Types):**

| Azure Series | AWS Equivalent | Purpose |
|--------------|----------------|---------|
| **B-series** | t3/t4g | Burstable, cost-effective |
| **D-series** | m5/m6i | General purpose |
| **E-series** | r5/r6i | Memory-optimized |
| **F-series** | c5/c6i | Compute-optimized |
| **N-series** | p3/p4 | GPU workloads |

**Create a VM:**

```bash
# Azure CLI
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# AWS CLI Equivalent
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.small \
  --key-name myKeyPair \
  --security-group-ids sg-xxxxx \
  --subnet-id subnet-xxxxx
```

### 3.3 VM Scale Sets (Auto Scaling)

```bash
# Create VMSS with autoscale
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image Ubuntu2204 \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys

# Configure autoscale rules
az monitor autoscale create \
  --resource-group myResourceGroup \
  --resource myScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add scale-out rule (CPU > 70%)
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1
```

**AWS ASG Equivalent:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-configuration-name my-lc \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-1,subnet-2"
```

### 3.4 Azure App Service (PaaS)

**Deployment Slots** = Blue/Green Deployments

```bash
# Create App Service Plan (like Beanstalk Environment Tier)
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myUniqueWebApp \
  --runtime "NODE|18-lts"

# Create staging slot
az webapp deployment slot create \
  --name myUniqueWebApp \
  --resource-group myResourceGroup \
  --slot staging

# Deploy code
az webapp deployment source config \
  --name myUniqueWebApp \
  --resource-group myResourceGroup \
  --repo-url https://github.com/user/repo \
  --branch main

# Swap staging to production (zero downtime)
az webapp deployment slot swap \
  --name myUniqueWebApp \
  --resource-group myResourceGroup \
  --slot staging
```

**App Service Plans (Tiers):**

| Azure Tier | AWS Equivalent | Features |
|------------|----------------|----------|
| **Free/Shared** | Free Tier | Limited resources |
| **Basic** | Single Instance | No autoscale, custom domains |
| **Standard** | Load Balanced | Autoscale, staging slots |
| **Premium** | Enhanced | More performance, VNet integration |
| **Isolated** | Dedicated Environment | App Service Environment (ASE) |

---

## 4. Storage Services

### 4.1 Storage Services Comparison

| Azure | AWS | Use Case |
|-------|-----|----------|
| **Blob Storage** | S3 | Object storage |
| **Files (SMB/NFS)** | EFS | Managed file shares |
| **Queue Storage** | SQS | Message queuing |
| **Table Storage** | DynamoDB | NoSQL key-value |
| **Disk Storage** | EBS | Block storage for VMs |
| **Data Lake Storage Gen2** | S3 + Lake Formation | Big data analytics |

### 4.2 Blob Storage Deep Dive

**Storage Account = AWS Account-level S3 configuration**
**Container = S3 Bucket**
**Blob = S3 Object**

**Access Tiers (Lifecycle):**

| Azure Tier | AWS Equivalent | Cost | Use Case |
|------------|----------------|------|----------|
| **Hot** | S3 Standard | Highest | Frequent access |
| **Cool** | S3 Standard-IA | Medium | Infrequent (30+ days) |
| **Cold** | S3 Glacier Instant | Lower | Rare access (90+ days) |
| **Archive** | S3 Glacier Deep Archive | Lowest | Long-term (180+ days) |

**Create Storage Account:**

```bash
# Azure CLI
az storage account create \
  --name mystorageaccount123 \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Create container (bucket)
az storage container create \
  --account-name mystorageaccount123 \
  --name mycontainer \
  --auth-mode login

# Upload blob
az storage blob upload \
  --account-name mystorageaccount123 \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./local-file.txt \
  --auth-mode login

# AWS S3 Equivalent
aws s3 mb s3://my-bucket
aws s3 cp local-file.txt s3://my-bucket/myfile.txt
```

**Redundancy Options:**

| Azure | AWS Equivalent | Description |
|-------|----------------|-------------|
| **LRS** (Locally Redundant) | S3 One Zone-IA | 3 copies in one datacenter |
| **ZRS** (Zone Redundant) | S3 Standard | 3 copies across AZs |
| **GRS** (Geo-Redundant) | S3 Cross-Region Replication | Replicated to paired region |
| **GZRS** (Geo-Zone Redundant) | S3 + CRR with Multi-AZ | Best durability |

### 4.3 Shared Access Signatures (SAS) vs S3 Pre-signed URLs

```bash
# Generate SAS token (like S3 pre-signed URL)
az storage blob generate-sas \
  --account-name mystorageaccount123 \
  --container-name mycontainer \
  --name myfile.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z \
  --auth-mode login

# AWS equivalent
aws s3 presign s3://my-bucket/myfile.txt --expires-in 3600
```

**SAS Types:**
- **Account SAS** = Account-level access
- **Service SAS** = Service-level (Blob, Queue, Table, File)
- **User Delegation SAS** = Azure AD-based (most secure)

### 4.4 Azure Files (SMB/NFS Shares)

```bash
# Create file share
az storage share create \
  --account-name mystorageaccount123 \
  --name myfileshare \
  --quota 100

# Mount on Linux
sudo mkdir /mnt/azurefiles
sudo mount -t cifs \
  //mystorageaccount123.file.core.windows.net/myfileshare \
  /mnt/azurefiles \
  -o username=mystorageaccount123,password=<key>,serverino

# Mount on Windows
net use Z: \\mystorageaccount123.file.core.windows.net\myfileshare /u:Azure\mystorageaccount123 <key>
```

**AWS EFS Equivalent:**
```bash
sudo mount -t nfs4 \
  fs-12345678.efs.us-east-1.amazonaws.com:/ /mnt/efs
```

---

## 5. Networking

### 5.1 Networking Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Virtual Network (VNet)** | VPC | Isolated network |
| **Subnet** | Subnet | Network segment |
| **Network Security Group (NSG)** | Security Group + NACL | Firewall rules |
| **Application Security Group (ASG)** | *Tags in SG* | Logical grouping |
| **Azure Load Balancer** | Network Load Balancer | Layer 4 load balancing |
| **Application Gateway** | Application Load Balancer | Layer 7 load balancing |
| **Azure Firewall** | AWS Network Firewall | Managed firewall |
| **VPN Gateway** | Virtual Private Gateway | Site-to-Site VPN |
| **ExpressRoute** | Direct Connect | Dedicated connection |
| **Azure Front Door** | CloudFront + Global Accelerator | CDN + Global LB |
| **Traffic Manager** | Route 53 Traffic Policies | DNS-based load balancing |
| **Azure DNS** | Route 53 | DNS service |
| **Private Endpoint** | PrivateLink | Private connectivity |
| **Service Endpoint** | Gateway VPC Endpoint | VNet to Azure services |

### 5.2 Virtual Network (VNet) Setup

```bash
# Create VNet
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.1.0/24

# Add subnet
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name myPrivateSubnet \
  --address-prefix 10.0.2.0/24

# AWS VPC Equivalent
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24
```

### 5.3 Network Security Groups (NSG)

**NSG = Security Group + Network ACL combined**

```bash
# Create NSG
az network nsg create \
  --resource-group myResourceGroup \
  --name myNSG

# Add rule (allow SSH)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowSSH \
  --priority 1000 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound

# Associate with subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --network-security-group myNSG
```

**Key Differences from AWS:**
- NSG can attach to **subnet** (like NACL) OR **NIC** (like Security Group)
- Rules have **priority** (100-4096, lower = higher priority)
- **Stateful** like Security Groups
- **Service Tags** (like AWS prefix lists): `Internet`, `VirtualNetwork`, `AzureLoadBalancer`, `Storage`, `Sql`

### 5.4 Application Security Groups (ASG)

```bash
# Create ASG (logical grouping)
az network asg create \
  --resource-group myResourceGroup \
  --name webServersASG

az network asg create \
  --resource-group myResourceGroup \
  --name dbServersASG

# Create NSG rule using ASG
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowWebToDb \
  --priority 1100 \
  --source-asgs webServersASG \
  --destination-asgs dbServersASG \
  --destination-port-ranges 3306 \
  --access Allow \
  --protocol Tcp

# Assign VM NIC to ASG
az network nic ip-config update \
  --resource-group myResourceGroup \
  --nic-name myNIC \
  --name ipconfig1 \
  --application-security-groups webServersASG
```

**AWS Equivalent:** Security Group rules with instance IDs or using tags (less elegant)

### 5.5 Load Balancers

**Azure Load Balancer (Layer 4):**

```bash
# Create public load balancer
az network lb create \
  --resource-group myResourceGroup \
  --name myLoadBalancer \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Add health probe
az network lb probe create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Add load balancing rule
az network lb rule create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myHTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe
```

**Application Gateway (Layer 7 - like AWS ALB):**

```bash
# Create Application Gateway
az network application-gateway create \
  --name myAppGateway \
  --resource-group myResourceGroup \
  --location eastus \
  --capacity 2 \
  --sku Standard_v2 \
  --vnet-name myVNet \
  --subnet mySubnet \
  --public-ip-address myAGPublicIP \
  --http-settings-cookie-based-affinity Enabled \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --servers 10.0.1.4 10.0.1.5
```

**Features:**
- URL path-based routing
- SSL termination
- WAF (Web Application Firewall)
- Cookie-based session affinity
- Autoscaling

### 5.6 VNet Peering (like VPC Peering)

```bash
# Peer VNet1 to VNet2
az network vnet peering create \
  --resource-group myResourceGroup \
  --name VNet1ToVNet2 \
  --vnet-name VNet1 \
  --remote-vnet VNet2 \
  --allow-vnet-access

# Peer VNet2 to VNet1 (bidirectional required)
az network vnet peering create \
  --resource-group myResourceGroup \
  --name VNet2ToVNet1 \
  --vnet-name VNet2 \
  --remote-vnet VNet1 \
  --allow-vnet-access
```

**Key Differences:**
- **Non-transitive** (same as AWS)
- **Cross-region** peering supported (Global VNet Peering)
- **No overlapping IP ranges** allowed

### 5.7 Private Endpoints vs Service Endpoints

**Service Endpoint (like VPC Gateway Endpoint):**
```bash
# Enable service endpoint on subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --service-endpoints Microsoft.Storage Microsoft.Sql
```

**Private Endpoint (like AWS PrivateLink):**
```bash
# Create private endpoint for Storage Account
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account} \
  --group-id blob \
  --connection-name myConnection
```

**Comparison:**

| Feature | Service Endpoint | Private Endpoint |
|---------|-----------------|------------------|
| **IP Address** | Public (Azure backbone) | Private VNet IP |
| **Scope** | All instances of service | Specific resource |
| **Cost** | Free | Paid (per endpoint/hour) |
| **On-premises** | No | Yes (via VPN/ExpressRoute) |

---

## 6. Databases

### 6.1 Database Services Comparison

| Azure | AWS | Type |
|-------|-----|------|
| **Azure SQL Database** | RDS for SQL Server | Managed SQL Server |
| **Azure Database for MySQL** | RDS for MySQL | Managed MySQL |
| **Azure Database for PostgreSQL** | RDS for PostgreSQL | Managed PostgreSQL |
| **Azure Cosmos DB** | DynamoDB | NoSQL multi-model |
| **Azure Cache for Redis** | ElastiCache for Redis | In-memory cache |
| **Azure Synapse Analytics** | Redshift | Data warehouse |
| **Azure Table Storage** | DynamoDB (basic) | NoSQL key-value |

### 6.2 Azure SQL Database

**Deployment Options:**
1. **Single Database** = RDS Single Instance
2. **Elastic Pool** = Shared resources across DBs
3. **Managed Instance** = Near 100% SQL Server compatibility

**Purchasing Models:**

| Model | AWS Equivalent | Description |
|-------|----------------|-------------|
| **DTU** (Database Transaction Unit) | *N/A* | Bundled compute/storage/IO |
| **vCore** | RDS Instance Types | Choose CPU/RAM separately |

**Create SQL Database:**

```bash
# Create SQL server (logical container)
az sql server create \
  --name mysqlserver123 \
  --resource-group myResourceGroup \
  --location eastus \
  --admin-user sqladmin \
  --admin-password P@ssw0rd1234!

# Configure firewall
az sql server firewall-rule create \
  --resource-group myResourceGroup \
  --server mysqlserver123 \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create database
az sql db create \
  --resource-group myResourceGroup \
  --server mysqlserver123 \
  --name myDatabase \
  --service-objective S0 \
  --backup-storage-redundancy Zone

# AWS RDS Equivalent
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine sqlserver-ex \
  --master-username admin \
  --master-user-password password \
  --allocated-storage 20
```

**Connection String:**
```
Server=tcp:mysqlserver123.database.windows.net,1433;
Database=myDatabase;
User ID=sqladmin;
Password=P@ssw0rd1234!;
Encrypt=yes;
```

### 6.3 Azure Cosmos DB (Multi-Model NoSQL)

**APIs (Models):**
1. **Core (SQL)** - Document database (like MongoDB)
2. **MongoDB** - Wire protocol compatible
3. **Cassandra** - Column-family
4. **Gremlin** - Graph database
5. **Table** - Key-value (Azure Table Storage compatible)

**Unique Features:**
- **Global Distribution** - Multi-region writes
- **Turnkey replication** - Click to add region
- **5 Consistency Levels** (Strong, Bounded Staleness, Session, Consistent Prefix, Eventual)
- **99.999% SLA** for multi-region

```bash
# Create Cosmos DB account
az cosmosdb create \
  --name mycosmosdb123 \
  --resource-group myResourceGroup \
  --kind GlobalDocumentDB \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1 \
  --default-consistency-level Session \
  --enable-multiple-write-locations true

# Create database
az cosmosdb sql database create \
  --account-name mycosmosdb123 \
  --resource-group myResourceGroup \
  --name myDatabase

# Create container (table)
az cosmosdb sql container create \
  --account-name mycosmosdb123 \
  --resource-group myResourceGroup \
  --database-name myDatabase \
  --name myContainer \
  --partition-key-path "/userId" \
  --throughput 400
```

**Pricing Models:**
- **Provisioned Throughput** (RU/s) = DynamoDB Provisioned
- **Serverless** = DynamoDB On-Demand
- **Autoscale** = DynamoDB Auto Scaling

**Code Example (Python):**

```python
from azure.cosmos import CosmosClient, PartitionKey

# Initialize client
client = CosmosClient(
    url="https://mycosmosdb123.documents.azure.com:443/",
    credential="<key>"
)

database = client.get_database_client("myDatabase")
container = database.get_container_client("myContainer")

# Insert item
container.upsert_item({
    "id": "1",
    "userId": "user123",
    "name": "John Doe",
    "email": "john@example.com"
})

# Query items
for item in container.query_items(
    query="SELECT * FROM c WHERE c.userId=@userId",
    parameters=[{"name": "@userId", "value": "user123"}],
    partition_key="user123"
):
    print(item)
```

**DynamoDB Equivalent (boto3):**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('myTable')

# Put item
table.put_item(Item={
    'userId': 'user123',
    'name': 'John Doe',
    'email': 'john@example.com'
})

# Query
response = table.query(
    KeyConditionExpression='userId = :userId',
    ExpressionAttributeValues={':userId': 'user123'}
)
```

### 6.4 Azure Database for PostgreSQL/MySQL

**Deployment Options:**
1. **Single Server** - Basic managed service (retiring)
2. **Flexible Server** - Enhanced features (recommended)
3. **Hyperscale (Citus)** - Distributed PostgreSQL (unique to Azure)

```bash
# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group myResourceGroup \
  --name mypostgresserver \
  --location eastus \
  --admin-user myadmin \
  --admin-password P@ssw0rd1234! \
  --sku-name Standard_B2s \
  --tier Burstable \
  --storage-size 32 \
  --version 14 \
  --high-availability Disabled

# Create database
az postgres flexible-server db create \
  --resource-group myResourceGroup \
  --server-name mypostgresserver \
  --database-name mydb
```

**High Availability:**
- **Zone-redundant HA** - Standby in different AZ (like RDS Multi-AZ)
- **Same-zone HA** - Standby in same AZ
- **Read replicas** - Read scaling (like RDS Read Replicas)

---

## 7. Containers & Orchestration

### 7.1 Container Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure Container Registry (ACR)** | ECR | Private container registry |
| **Azure Container Instances (ACI)** | Fargate (standalone) | Serverless containers |
| **Azure Kubernetes Service (AKS)** | EKS | Managed Kubernetes |
| **Azure Container Apps** | App Runner | Serverless containers (PaaS) |
| **Azure Red Hat OpenShift** | *N/A* | Managed OpenShift |

### 7.2 Azure Container Registry (ACR)

```bash
# Create container registry
az acr create \
  --resource-group myResourceGroup \
  --name mycontainerregistry123 \
  --sku Premium \
  --location eastus

# Enable admin account (for basic auth)
az acr update \
  --name mycontainerregistry123 \
  --admin-enabled true

# Login
az acr login --name mycontainerregistry123

# Tag and push image
docker tag myapp:latest mycontainerregistry123.azurecr.io/myapp:v1
docker push mycontainerregistry123.azurecr.io/myapp:v1

# AWS ECR Equivalent
aws ecr create-repository --repository-name myapp
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
docker push <account>.dkr.ecr.<region>.amazonaws.com/myapp:v1
```

**SKU Tiers:**
- **Basic** = Lower throughput
- **Standard** = Production use
- **Premium** = Geo-replication, content trust, private endpoints

### 7.3 Azure Kubernetes Service (AKS)

```bash
# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity \
  --network-plugin azure \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --attach-acr mycontainerregistry123

# Get credentials
az aks get-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Verify
kubectl get nodes

# AWS EKS Equivalent
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3
```

**AKS Unique Features:**
- **Virtual Nodes** - Burst to ACI (like Fargate for EKS)
- **HTTP Application Routing** - Automatic ingress DNS
- **Azure Policy** - Built-in governance
- **Free control plane** (AWS charges $0.10/hour)
- **Automatic upgrades** - Hands-free K8s version management

**Enable Autoscaling:**

```bash
# Cluster autoscaler
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10

# Horizontal Pod Autoscaler (HPA)
kubectl autoscale deployment myapp \
  --cpu-percent=50 \
  --min=1 \
  --max=10
```

### 7.4 Azure Container Instances (ACI)

**Fastest way to run a container (no orchestration needed):**

```bash
# Run container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --dns-name-label myapp-dns \
  --ports 80 \
  --cpu 1 \
  --memory 1.5

# View logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# Execute command
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

**Use Cases:**
- CI/CD build agents
- Batch jobs
- Dev/test environments
- Burst compute from AKS

**Integration with AKS (Virtual Nodes):**

```yaml
# Pod with node selector for ACI
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: mycontainerregistry123.azurecr.io/myapp:v1
  nodeSelector:
    kubernetes.io/role: agent
    type: virtual-kubelet
  tolerations:
  - key: virtual-kubelet.io/provider
    operator: Exists
```

### 7.5 Azure Container Apps (Serverless PaaS)

**Like AWS App Runner + Lambda combined:**

```bash
# Create Container Apps environment
az containerapp env create \
  --name myenv \
  --resource-group myResourceGroup \
  --location eastus

# Deploy container app
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenv \
  --image mycontainerregistry123.azurecr.io/myapp:v1 \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10 \
  --cpu 0.5 \
  --memory 1.0Gi
```

**Features:**
- **Scale to zero** (like Lambda)
- **KEDA-powered autoscaling** (event-driven)
- **Dapr integration** (microservices framework)
- **Revisions** (versioning like App Service slots)
- **Traffic splitting** (A/B testing)

---

## 8. Serverless & Event-Driven Architecture

### 8.1 Serverless Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure Functions** | Lambda | Serverless functions |
| **Logic Apps** | Step Functions | Workflow orchestration |
| **Event Grid** | EventBridge | Event routing |
| **Service Bus** | SQS + SNS | Message broker |
| **Event Hubs** | Kinesis | Event streaming |
| **Durable Functions** | Step Functions (alternative) | Stateful workflows |

### 8.2 Azure Functions

**Hosting Plans:**

| Plan | AWS Equivalent | Description |
|------|----------------|-------------|
| **Consumption** | Lambda | Pay-per-execution, auto-scale |
| **Premium** | Lambda Provisioned | Pre-warmed instances, VNet |
| **Dedicated** | Lambda on EC2 | App Service Plan |

**Create Function App:**

```bash
# Create storage account (required for Functions)
az storage account create \
  --name myfunctionstorage123 \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS

# Create Function App
az functionapp create \
  --resource-group myResourceGroup \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.9 \
  --functions-version 4 \
  --name myfunctionapp123 \
  --storage-account myfunctionstorage123 \
  --os-type Linux
```

**Example Function (HTTP Trigger):**

```python
# __init__.py
import logging
import json
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
            name = req_body.get('name')
        except ValueError:
            pass

    if name:
        return func.HttpResponse(
            f"Hello, {name}!",
            status_code=200
        )
    else:
        return func.HttpResponse(
            "Please pass a name",
            status_code=400
        )
```

**function.json (configuration):**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

**AWS Lambda Equivalent:**

```python
import json

def lambda_handler(event, context):
    name = event.get('queryStringParameters', {}).get('name', '')

    if name:
        return {
            'statusCode': 200,
            'body': json.dumps(f'Hello, {name}!')
        }
    else:
        return {
            'statusCode': 400,
            'body': json.dumps('Please pass a name')
        }
```

**Triggers & Bindings:**

| Trigger Type | AWS Equivalent | Description |
|--------------|----------------|-------------|
| **HTTP** | API Gateway + Lambda | REST API |
| **Timer** | CloudWatch Events | Cron/scheduled |
| **Blob** | S3 + Lambda | Storage events |
| **Queue** | SQS + Lambda | Queue messages |
| **Event Hub** | Kinesis + Lambda | Streaming events |
| **Cosmos DB** | DynamoDB Streams | Database changes |
| **Service Bus** | SNS/SQS + Lambda | Message broker |

**Example: Blob Trigger Function**

```python
# function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "myblob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "mycontainer/{name}",
      "connection": "AzureWebJobsStorage"
    }
  ]
}

# __init__.py
import logging
import azure.functions as func

def main(myblob: func.InputStream):
    logging.info(f"Blob trigger function processed blob \n"
                 f"Name: {myblob.name}\n"
                 f"Blob Size: {myblob.length} bytes")

    content = myblob.read()
    # Process blob content
```

### 8.3 Durable Functions (Stateful Workflows)

**Unique to Azure - No direct AWS equivalent (similar to Step Functions but code-based)**

**Patterns:**
1. **Function Chaining** - Sequential execution
2. **Fan-out/Fan-in** - Parallel execution
3. **Async HTTP APIs** - Long-running operations
4. **Monitoring** - Recurring process
5. **Human Interaction** - Approval workflows

**Example: Function Chaining**

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    # Chain functions
    x = yield context.call_activity('Step1', None)
    y = yield context.call_activity('Step2', x)
    z = yield context.call_activity('Step3', y)
    return z

main = df.Orchestrator.create(orchestrator_function)
```

**Example: Fan-out/Fan-in**

```python
def orchestrator_function(context: df.DurableOrchestrationContext):
    files = yield context.call_activity('GetFiles', None)

    # Process files in parallel
    tasks = [context.call_activity('ProcessFile', file) for file in files]
    results = yield context.task_all(tasks)

    # Aggregate results
    summary = yield context.call_activity('CreateSummary', results)
    return summary

main = df.Orchestrator.create(orchestrator_function)
```

### 8.4 Event Grid (Event Routing)

**Like AWS EventBridge - Event bus for Azure services**

```bash
# Create custom topic
az eventgrid topic create \
  --name myeventtopic \
  --resource-group myResourceGroup \
  --location eastus

# Subscribe to events (webhook)
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/myeventtopic \
  --endpoint https://myapp.azurewebsites.net/api/webhook
```

**Publish Event:**

```python
from azure.eventgrid import EventGridPublisherClient
from azure.core.credentials import AzureKeyCredential
from azure.eventgrid import EventGridEvent

client = EventGridPublisherClient(
    endpoint="https://myeventtopic.eastus-1.eventgrid.azure.net/api/events",
    credential=AzureKeyCredential("<key>")
)

event = EventGridEvent(
    event_type="MyApp.OrderCreated",
    data={"orderId": "12345", "amount": 99.99},
    subject="orders/12345",
    data_version="1.0"
)

client.send([event])
```

**Built-in Event Sources:**
- Blob Storage (create, delete)
- Resource Groups (resource changes)
- Azure subscriptions
- Event Hubs
- IoT Hub
- Key Vault
- Container Registry

### 8.5 Service Bus (Enterprise Messaging)

**vs AWS SQS + SNS:**

| Feature | Service Bus | SQS + SNS |
|---------|-------------|-----------|
| **Max message size** | 256 KB (1 MB Premium) | 256 KB |
| **Retention** | Up to 7 days (unlimited Premium) | Up to 14 days |
| **Ordering** | Sessions (guaranteed) | FIFO queues |
| **Topics** | Built-in pub/sub | SNS required |
| **Transactions** | Yes | No |
| **Duplicate detection** | Yes | No |

```bash
# Create Service Bus namespace
az servicebus namespace create \
  --resource-group myResourceGroup \
  --name myservicebus123 \
  --location eastus \
  --sku Standard

# Create queue
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myservicebus123 \
  --name myqueue \
  --max-size 1024 \
  --default-message-time-to-live P14D

# Create topic (pub/sub)
az servicebus topic create \
  --resource-group myResourceGroup \
  --namespace-name myservicebus123 \
  --name mytopic

# Create subscription
az servicebus topic subscription create \
  --resource-group myResourceGroup \
  --namespace-name myservicebus123 \
  --topic-name mytopic \
  --name mysubscription
```

**Code Example:**

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

# Send message
conn_str = "<connection-string>"
client = ServiceBusClient.from_connection_string(conn_str)
sender = client.get_queue_sender("myqueue")

with sender:
    message = ServiceBusMessage("Hello, Service Bus!")
    sender.send_messages(message)

# Receive message
receiver = client.get_queue_receiver("myqueue")
with receiver:
    messages = receiver.receive_messages(max_message_count=10, max_wait_time=5)
    for message in messages:
        print(str(message))
        receiver.complete_message(message)
```

---

## 9. Monitoring & Management

### 9.1 Monitoring Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure Monitor** | CloudWatch | Metrics & logs |
| **Application Insights** | X-Ray + CloudWatch | APM & tracing |
| **Log Analytics** | CloudWatch Logs Insights | Log querying |
| **Azure Alerts** | CloudWatch Alarms | Alerting |
| **Azure Service Health** | Personal Health Dashboard | Service status |
| **Azure Advisor** | Trusted Advisor | Recommendations |

### 9.2 Azure Monitor & Log Analytics

**Enable diagnostics:**

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group myResourceGroup \
  --workspace-name mylogworkspace

# Enable diagnostics for a VM
az monitor diagnostic-settings create \
  --name myDiagSettings \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/myVM \
  --workspace mylogworkspace \
  --metrics '[{"category": "AllMetrics","enabled": true}]' \
  --logs '[{"category": "Administrative","enabled": true}]'
```

**KQL (Kusto Query Language) - like CloudWatch Logs Insights:**

```kql
// View all logs from last hour
AzureDiagnostics
| where TimeGenerated > ago(1h)
| project TimeGenerated, ResourceType, Resource, Category, OperationName

// CPU usage over time
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart

// Failed HTTP requests
requests
| where success == false
| summarize count() by resultCode, bin(timestamp, 1h)
| render barchart

// Application errors
traces
| where severityLevel >= 3
| project timestamp, message, severityLevel, operation_Name
| order by timestamp desc
```

### 9.3 Application Insights (APM)

**Automatically instrument your app:**

```bash
# Create Application Insights
az monitor app-insights component create \
  --app myAppInsights \
  --resource-group myResourceGroup \
  --location eastus \
  --application-type web

# Get instrumentation key
az monitor app-insights component show \
  --app myAppInsights \
  --resource-group myResourceGroup \
  --query instrumentationKey
```

**Python Example:**

```python
from opencensus.ext.azure import metrics_exporter
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

# Configure Application Insights
tracer = Tracer(
    exporter=AzureExporter(
        connection_string='InstrumentationKey=<key>'
    ),
    sampler=ProbabilitySampler(1.0),
)

# Trace function
with tracer.span(name='ProcessOrder'):
    # Your code here
    process_order()
```

**.NET Example:**

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;

var telemetryClient = new TelemetryClient();

// Track event
telemetryClient.TrackEvent("OrderPlaced",
    new Dictionary<string, string> { { "OrderId", "12345" } });

// Track exception
try {
    ProcessOrder();
} catch (Exception ex) {
    telemetryClient.TrackException(ex);
    throw;
}

// Track dependency
using (var operation = telemetryClient.StartOperation<DependencyTelemetry>("DatabaseQuery")) {
    operation.Telemetry.Type = "SQL";
    operation.Telemetry.Data = "SELECT * FROM Orders";
    ExecuteQuery();
}
```

**Features:**
- **Live Metrics** - Real-time monitoring
- **Application Map** - Dependency visualization
- **Smart Detection** - Anomaly detection
- **Profiler** - Performance profiling
- **Snapshot Debugger** - Production debugging

### 9.4 Alerts

```bash
# Create metric alert (CPU > 80%)
az monitor metrics alert create \
  --name HighCPU \
  --resource-group myResourceGroup \
  --scopes /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/myVM \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-group myActionGroup

# Create log alert
az monitor scheduled-query create \
  --name FailedRequests \
  --resource-group myResourceGroup \
  --scopes /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/myAppInsights \
  --condition "count > 10" \
  --condition-query "requests | where success == false | summarize count()" \
  --evaluation-frequency 5m \
  --window-size 5m \
  --action-groups myActionGroup
```

---

## 10. DevOps & CI/CD

### 10.1 DevOps Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure DevOps** | CodeCommit + CodeBuild + CodeDeploy + CodePipeline | Complete DevOps platform |
| **Azure Repos** | CodeCommit | Git repositories |
| **Azure Pipelines** | CodePipeline | CI/CD pipelines |
| **Azure Artifacts** | CodeArtifact | Package management |
| **Azure Boards** | *N/A* | Agile planning |
| **Azure Test Plans** | Device Farm | Testing |

### 10.2 Azure Pipelines

**azure-pipelines.yml (like AWS buildspec.yml + appspec.yml):**

```yaml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: UseDotNet@2
      inputs:
        version: '6.x'

    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'

    - task: DotNetCoreCLI@2
      displayName: 'Build project'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'

    - task: DotNetCoreCLI@2
      displayName: 'Run tests'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration) --no-build'

    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: DeployWeb
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyAzureSubscription'
              appType: 'webAppLinux'
              appName: 'mywebapp'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

**Docker Build & Push to ACR:**

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'myACRConnection'
  imageRepository: 'myapp'
  containerRegistry: 'mycontainerregistry123.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: Build and push image
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: Docker@2
      displayName: Build and push image
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest

- stage: Deploy
  displayName: Deploy to AKS
  dependsOn: Build
  jobs:
  - deployment: Deploy
    displayName: Deploy
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: Deploy to Kubernetes
            inputs:
              action: deploy
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yml
                $(Pipeline.Workspace)/manifests/service.yml
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)
```

### 10.3 GitHub Actions with Azure

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

env:
  AZURE_WEBAPP_NAME: mywebapp
  AZURE_WEBAPP_PACKAGE_PATH: '.'
  NODE_VERSION: '18.x'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: npm install and build
      run: |
        npm install
        npm run build --if-present

    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

    - name: Logout
      run: az logout
```

**Deploy to AKS:**

```yaml
name: Deploy to AKS

on:
  push:
    branches: [ main ]

env:
  REGISTRY_NAME: mycontainerregistry123
  CLUSTER_NAME: myAKSCluster
  CLUSTER_RESOURCE_GROUP: myResourceGroup
  NAMESPACE: production
  APP_NAME: myapp

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Build and push image
      run: |
        az acr build --registry ${{ env.REGISTRY_NAME }} \
          --image ${{ env.APP_NAME }}:${{ github.sha }} \
          --image ${{ env.APP_NAME }}:latest \
          .

    - name: Set AKS context
      uses: azure/aks-set-context@v3
      with:
        resource-group: ${{ env.CLUSTER_RESOURCE_GROUP }}
        cluster-name: ${{ env.CLUSTER_NAME }}

    - name: Deploy to AKS
      uses: azure/k8s-deploy@v4
      with:
        namespace: ${{ env.NAMESPACE }}
        manifests: |
          manifests/deployment.yml
          manifests/service.yml
        images: |
          ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
```

### 10.4 ARM Template Deployment in Pipeline

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'MyServiceConnection'
    subscriptionId: '$(subscriptionId)'
    action: 'Create Or Update Resource Group'
    resourceGroupName: '$(resourceGroupName)'
    location: 'East US'
    templateLocation: 'Linked artifact'
    csmFile: '$(Build.SourcesDirectory)/templates/azuredeploy.json'
    csmParametersFile: '$(Build.SourcesDirectory)/templates/azuredeploy.parameters.json'
    deploymentMode: 'Incremental'
```

---

## 11. Security & Compliance

### 11.1 Security Services Comparison

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Azure Key Vault** | Secrets Manager + KMS | Secrets & key management |
| **Azure Security Center (Defender)** | Security Hub | Security posture |
| **Azure Sentinel** | Security Lake + GuardDuty | SIEM |
| **Azure DDoS Protection** | Shield | DDoS mitigation |
| **Azure Firewall** | Network Firewall | Managed firewall |
| **Azure WAF** | WAF | Web application firewall |
| **Azure Policy** | Config Rules | Governance |
| **Azure Blueprints** | Service Catalog | Environment templates |

### 11.2 Azure Key Vault

```bash
# Create Key Vault
az keyvault create \
  --name mykeyvault123 \
  --resource-group myResourceGroup \
  --location eastus \
  --enable-rbac-authorization false

# Store secret
az keyvault secret set \
  --vault-name mykeyvault123 \
  --name DatabasePassword \
  --value 'SuperSecretP@ssw0rd!'

# Retrieve secret
az keyvault secret show \
  --vault-name mykeyvault123 \
  --name DatabasePassword \
  --query value -o tsv

# Create encryption key
az keyvault key create \
  --vault-name mykeyvault123 \
  --name myEncryptionKey \
  --protection software

# Store certificate
az keyvault certificate import \
  --vault-name mykeyvault123 \
  --name myCert \
  --file cert.pfx \
  --password certPassword
```

**Access from Code (Python):**

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://mykeyvault123.vault.azure.net/",
    credential=credential
)

# Get secret
secret = client.get_secret("DatabasePassword")
print(f"Secret value: {secret.value}")

# Set secret
client.set_secret("ApiKey", "abc123xyz")
```

**Access Policies vs RBAC:**

```bash
# Access Policy approach (legacy)
az keyvault set-policy \
  --name mykeyvault123 \
  --upn user@example.com \
  --secret-permissions get list set delete

# RBAC approach (recommended)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee user@example.com \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault123
```

### 11.3 Managed Identity with Key Vault

```bash
# Grant VM's managed identity access to Key Vault
az keyvault set-policy \
  --name mykeyvault123 \
  --object-id <managed-identity-object-id> \
  --secret-permissions get list
```

**Code on VM (no credentials needed):**

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Automatically uses VM's managed identity
credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://mykeyvault123.vault.azure.net/",
    credential=credential
)

secret = client.get_secret("DatabasePassword")
```

### 11.4 Azure Policy

**Like AWS Config Rules but proactive (can deny deployments):**

```bash
# Assign built-in policy (require tags)
az policy assignment create \
  --name RequireTags \
  --policy /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62 \
  --scope /subscriptions/{subscription-id} \
  --params '{"tagName": {"value": "Environment"}}'

# Create custom policy
az policy definition create \
  --name AllowedLocations \
  --description "Restricts resource locations" \
  --rules '{
    "if": {
      "not": {
        "field": "location",
        "in": ["eastus", "westus"]
      }
    },
    "then": {
      "effect": "deny"
    }
  }'
```

**Example Custom Policy (JSON):**

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "not": {
            "field": "Microsoft.Storage/storageAccounts/enableHttpsTrafficOnly",
            "equals": "true"
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

### 11.5 Azure Defender for Cloud (Security Center)

```bash
# Enable Defender for specific resource types
az security pricing create \
  --name VirtualMachines \
  --tier Standard

az security pricing create \
  --name SqlServers \
  --tier Standard

az security pricing create \
  --name StorageAccounts \
  --tier Standard

az security pricing create \
  --name KubernetesService \
  --tier Standard
```

**Features:**
- **Secure Score** - Security posture rating
- **Recommendations** - Remediation guidance
- **Threat Protection** - Runtime protection
- **Regulatory Compliance** - Compliance dashboards
- **Workflow Automation** - Logic Apps integration

---

## 12. Cost Management

### 12.1 Cost Management Tools

| Azure | AWS | Purpose |
|-------|-----|---------|
| **Cost Management + Billing** | Cost Explorer | Cost analysis |
| **Azure Advisor** | Trusted Advisor | Recommendations |
| **Azure Budgets** | Budgets | Budget alerts |
| **Azure Reservations** | Reserved Instances | Commitment discounts |
| **Azure Hybrid Benefit** | *N/A* | License portability |
| **Spot VMs** | Spot Instances | Discounted compute |

### 12.2 Cost Analysis & Budgets

```bash
# Create budget
az consumption budget create \
  --budget-name MyBudget \
  --amount 1000 \
  --category Cost \
  --time-grain Monthly \
  --start-date 2024-01-01 \
  --end-date 2024-12-31 \
  --notifications '{
    "actual_GreaterThan_80": {
      "enabled": true,
      "operator": "GreaterThan",
      "threshold": 80,
      "contactEmails": ["admin@example.com"],
      "contactRoles": ["Owner", "Contributor"]
    }
  }'
```

**PowerShell Cost Query:**

```powershell
# Get cost by resource group
$costs = Get-AzConsumptionUsageDetail -StartDate 2024-01-01 -EndDate 2024-01-31
$costs | Group-Object ResourceGroup | Select-Object Name, @{
    Name="TotalCost";
    Expression={($_.Group | Measure-Object -Property PretaxCost -Sum).Sum}
}
```

### 12.3 Azure Reservations

```bash
# Purchase 1-year VM reservation (save up to 72%)
az reservations reservation-order purchase \
  --reservation-order-id <order-id> \
  --sku Standard_D2s_v3 \
  --location eastus \
  --reserved-resource-type VirtualMachines \
  --billing-scope /subscriptions/{subscription-id} \
  --quantity 10 \
  --term P1Y \
  --billing-plan Monthly

# List reservations
az reservations reservation-order list
```

**Reservation Types:**
- **VMs** (1 or 3 years) - Up to 72% savings
- **SQL Database** (1 or 3 years) - Up to 80% savings
- **Cosmos DB** (1 or 3 years) - Up to 65% savings
- **Storage** (1 or 3 years) - Up to 38% savings
- **App Service** (1 or 3 years) - Up to 55% savings

### 12.4 Spot VMs

```bash
# Create Spot VM (save up to 90%)
az vm create \
  --resource-group myResourceGroup \
  --name mySpotVM \
  --image Ubuntu2204 \
  --priority Spot \
  --max-price -1 \
  --eviction-policy Deallocate \
  --size Standard_D2s_v3
```

**Spot VM Scale Set:**

```bash
az vmss create \
  --resource-group myResourceGroup \
  --name mySpotVMSS \
  --image Ubuntu2204 \
  --priority Spot \
  --eviction-policy Delete \
  --max-price 0.05 \
  --instance-count 10 \
  --upgrade-policy-mode Automatic
```

### 12.5 Cost Optimization Tips

**Tagging Strategy:**

```bash
# Apply tags for cost allocation
az resource tag \
  --ids /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/myVM \
  --tags Environment=Production CostCenter=Engineering Project=WebApp

# Query costs by tag
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --query "[?tags.Environment=='Production']"
```

**Auto-shutdown VMs (Dev/Test):**

```bash
# Configure auto-shutdown
az vm auto-shutdown \
  --resource-group myResourceGroup \
  --name myDevVM \
  --time 1900 \
  --email admin@example.com
```

---

## 13. Advanced Architectures

### 13.1 Multi-Tier Web Application

```
Azure Front Door (Global LB + CDN)
    |
    v
Application Gateway (Regional L7 LB + WAF)
    |
    v
VM Scale Set (Web Tier)
    |
    v
Azure Load Balancer (Internal)
    |
    v
VM Scale Set (App Tier)
    |
    v
Azure SQL Database (Data Tier)
    |
    v
Azure Cache for Redis (Caching)
```

**Bicep Template:**

```bicep
// Virtual Network
resource vnet 'Microsoft.Network/virtualNetworks@2021-05-01' = {
  name: 'myVNet'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.0.0.0/16']
    }
    subnets: [
      {
        name: 'webSubnet'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
      {
        name: 'appSubnet'
        properties: {
          addressPrefix: '10.0.2.0/24'
        }
      }
      {
        name: 'dataSubnet'
        properties: {
          addressPrefix: '10.0.3.0/24'
        }
      }
    ]
  }
}

// Web Tier VMSS
resource webVmss 'Microsoft.Compute/virtualMachineScaleSets@2021-11-01' = {
  name: 'webVmss'
  location: resourceGroup().location
  sku: {
    name: 'Standard_D2s_v3'
    tier: 'Standard'
    capacity: 2
  }
  properties: {
    upgradePolicy: {
      mode: 'Automatic'
    }
    virtualMachineProfile: {
      osProfile: {
        computerNamePrefix: 'web'
        adminUsername: 'azureuser'
        linuxConfiguration: {
          disablePasswordAuthentication: true
          ssh: {
            publicKeys: [
              {
                path: '/home/azureuser/.ssh/authorized_keys'
                keyData: '<ssh-public-key>'
              }
            ]
          }
        }
      }
      storageProfile: {
        imageReference: {
          publisher: 'Canonical'
          offer: 'UbuntuServer'
          sku: '22.04-LTS'
          version: 'latest'
        }
      }
      networkProfile: {
        networkInterfaceConfigurations: [
          {
            name: 'webNic'
            properties: {
              primary: true
              ipConfigurations: [
                {
                  name: 'ipconfig'
                  properties: {
                    subnet: {
                      id: vnet.properties.subnets[0].id
                    }
                    loadBalancerBackendAddressPools: [
                      {
                        id: resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'webLB', 'webBackend')
                      }
                    ]
                  }
                }
              ]
            }
          }
        ]
      }
    }
  }
}

// Application Gateway
resource appGw 'Microsoft.Network/applicationGateways@2021-05-01' = {
  name: 'myAppGateway'
  location: resourceGroup().location
  properties: {
    sku: {
      name: 'WAF_v2'
      tier: 'WAF_v2'
      capacity: 2
    }
    // ... configuration
  }
}

// Azure SQL Database
resource sqlServer 'Microsoft.Sql/servers@2021-11-01' = {
  name: 'mysqlserver'
  location: resourceGroup().location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: '<password>'
  }
}

resource sqlDb 'Microsoft.Sql/servers/databases@2021-11-01' = {
  parent: sqlServer
  name: 'myDatabase'
  location: resourceGroup().location
  sku: {
    name: 'S1'
    tier: 'Standard'
  }
}
```

### 13.2 Serverless Event-Driven Architecture

```
Event Source (Blob Upload, HTTP, Queue)
    |
    v
Event Grid
    |
    v
Azure Function (Process)
    |
    v
Cosmos DB (Store)
    |
    v
Azure Function (Trigger on Change)
    |
    v
Service Bus (Notify)
```

**Example: Image Processing Pipeline**

```python
# function_app.py
import azure.functions as func
import logging
from azure.storage.blob import BlobServiceClient
from PIL import Image
import io

app = func.FunctionApp()

@app.blob_trigger(
    arg_name="myblob",
    path="uploads/{name}",
    connection="AzureWebJobsStorage"
)
@app.blob_output(
    arg_name="outputblob",
    path="thumbnails/{name}",
    connection="AzureWebJobsStorage"
)
def create_thumbnail(myblob: func.InputStream, outputblob: func.Out[bytes]):
    logging.info(f"Processing blob: {myblob.name}")

    # Read image
    image = Image.open(io.BytesIO(myblob.read()))

    # Create thumbnail
    image.thumbnail((200, 200))

    # Save to output
    output = io.BytesIO()
    image.save(output, format='JPEG')
    outputblob.set(output.getvalue())

@app.cosmos_db_trigger(
    arg_name="documents",
    database_name="mydb",
    collection_name="mycollection",
    connection_string_setting="CosmosDbConnection",
    lease_collection_name="leases",
    create_lease_collection_if_not_exists=True
)
@app.service_bus_queue_output(
    arg_name="msg",
    queue_name="notifications",
    connection="ServiceBusConnection"
)
def process_changes(documents: func.DocumentList, msg: func.Out[str]):
    for doc in documents:
        logging.info(f"Document changed: {doc['id']}")
        msg.set(f"Document {doc['id']} was updated")
```

### 13.3 Microservices on AKS

```
Azure Front Door
    |
    v
Application Gateway (Ingress)
    |
    v
AKS Cluster
    |- API Gateway (NGINX/Traefik)
    |- Service Mesh (Istio/Linkerd)
    |- Microservices (Pods)
    |   |- Service A
    |   |- Service B
    |   |- Service C
    |- Azure Monitor (Container Insights)
    |- Azure Key Vault (Secrets via CSI)
```

**Deployment Manifest:**

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
        version: v1
    spec:
      serviceAccountName: order-service-sa
      containers:
      - name: order-service
        image: mycontainerregistry123.azurecr.io/order-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_CONNECTION
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: production
spec:
  selector:
    app: order-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP

---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: tls-secret
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /orders/*
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

### 13.4 Data Analytics Pipeline

```
Data Sources (IoT Hub, Event Hubs, Blob)
    |
    v
Azure Data Factory (ETL)
    |
    v
Azure Data Lake Storage Gen2 (Raw)
    |
    v
Azure Databricks (Processing)
    |
    v
Azure Synapse Analytics (Warehouse)
    |
    v
Power BI (Visualization)
```

**Data Factory Pipeline (JSON):**

```json
{
  "name": "CopyBlobToDataLake",
  "properties": {
    "activities": [
      {
        "name": "CopyData",
        "type": "Copy",
        "inputs": [
          {
            "referenceName": "SourceBlobDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "DestDataLakeDataset",
            "type": "DatasetReference"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "AzureBlobFSSink"
          }
        }
      }
    ],
    "triggers": [
      {
        "name": "DailyTrigger",
        "type": "ScheduleTrigger",
        "typeProperties": {
          "recurrence": {
            "frequency": "Day",
            "interval": 1,
            "startTime": "2024-01-01T00:00:00Z"
          }
        }
      }
    ]
  }
}
```

---

## Quick Reference: Azure CLI Commands

```bash
# Login
az login

# Set subscription
az account set --subscription <subscription-id>

# List locations
az account list-locations -o table

# Resource Groups
az group create -n myRG -l eastus
az group list -o table
az group delete -n myRG --yes

# Virtual Machines
az vm create -g myRG -n myVM --image Ubuntu2204 --size Standard_B2s
az vm list -o table
az vm start -g myRG -n myVM
az vm stop -g myRG -n myVM
az vm deallocate -g myRG -n myVM

# App Service
az webapp create -g myRG -p myPlan -n myWebApp --runtime "NODE|18-lts"
az webapp list -o table
az webapp log tail -g myRG -n myWebApp

# Storage
az storage account create -n mystorage -g myRG -l eastus --sku Standard_LRS
az storage container create --account-name mystorage -n mycontainer
az storage blob upload --account-name mystorage -c mycontainer -f file.txt

# AKS
az aks create -g myRG -n myAKS --node-count 3
az aks get-credentials -g myRG -n myAKS
az aks scale -g myRG -n myAKS --node-count 5

# SQL Database
az sql server create -g myRG -n myserver -l eastus --admin-user admin --admin-password P@ss123
az sql db create -g myRG -s myserver -n mydb --service-objective S0

# Key Vault
az keyvault create -g myRG -n mykeyvault -l eastus
az keyvault secret set --vault-name mykeyvault -n MySecret --value SecretValue
az keyvault secret show --vault-name mykeyvault -n MySecret
```

---

## AWS to Azure Service Mapping (Quick Reference)

| AWS Service | Azure Service | Category |
|-------------|---------------|----------|
| EC2 | Virtual Machines | Compute |
| Lambda | Functions | Serverless |
| ECS/EKS | AKS | Containers |
| Fargate | Container Instances | Serverless Containers |
| Elastic Beanstalk | App Service | PaaS |
| S3 | Blob Storage | Object Storage |
| EBS | Managed Disks | Block Storage |
| EFS | Azure Files | File Storage |
| Glacier | Archive Storage | Cold Storage |
| VPC | Virtual Network | Networking |
| Route 53 | DNS / Traffic Manager | DNS |
| CloudFront | Front Door / CDN | CDN |
| ELB/ALB | Load Balancer / App Gateway | Load Balancing |
| Direct Connect | ExpressRoute | Hybrid Connection |
| RDS | SQL Database / Database for MySQL/PostgreSQL | Relational DB |
| DynamoDB | Cosmos DB | NoSQL |
| Redshift | Synapse Analytics | Data Warehouse |
| ElastiCache | Cache for Redis | Caching |
| SQS | Storage Queue / Service Bus | Queuing |
| SNS | Service Bus | Pub/Sub |
| EventBridge | Event Grid | Event Routing |
| Kinesis | Event Hubs | Streaming |
| Step Functions | Logic Apps / Durable Functions | Orchestration |
| CloudWatch | Azure Monitor | Monitoring |
| X-Ray | Application Insights | APM |
| IAM | Azure AD + RBAC | Identity |
| Secrets Manager | Key Vault | Secrets |
| KMS | Key Vault | Encryption Keys |
| CloudFormation | ARM / Bicep | IaC |
| CodePipeline | DevOps Pipelines | CI/CD |
| Organizations | Management Groups | Organization |
| Config | Policy | Governance |
| GuardDuty | Defender for Cloud | Threat Detection |
| WAF | Web Application Firewall | Application Security |

---

## Best Practices Summary

### 1. **Naming Conventions**
- Use consistent, descriptive names
- Follow Microsoft's recommended abbreviations
- Example: `rg-prod-eastus-001`, `vm-web-prod-001`

### 2. **Resource Organization**
- Group related resources in same Resource Group
- Use tags extensively for cost allocation
- Implement Management Groups for governance

### 3. **Security**
- Enable Managed Identities (avoid credentials)
- Use Azure AD authentication everywhere possible
- Implement RBAC with least privilege
- Enable encryption at rest and in transit
- Use Private Endpoints for PaaS services

### 4. **High Availability**
- Deploy across Availability Zones
- Use Zone-redundant storage
- Implement health probes
- Configure autoscaling

### 5. **Cost Optimization**
- Right-size VMs
- Use Reservations for predictable workloads
- Implement auto-shutdown for dev/test
- Leverage Spot VMs for fault-tolerant workloads
- Monitor with Cost Management

### 6. **Monitoring**
- Enable diagnostics on all resources
- Use Log Analytics workspaces
- Set up alerts proactively
- Implement Application Insights for apps

---

## Next Steps

1. **Get Certified**
   - AZ-900 (Fundamentals)
   - AZ-104 (Administrator)
   - AZ-204 (Developer)
   - AZ-305 (Architect)

2. **Hands-On Practice**
   - Create free Azure account ($200 credit)
   - Build sample applications
   - Complete Microsoft Learn modules

3. **Deep Dive Topics**
   - Azure landing zones
   - Azure Well-Architected Framework
   - Cloud Adoption Framework
   - Enterprise-scale architecture

---

## Resources

- **Official Documentation**: https://docs.microsoft.com/azure
- **Azure CLI Reference**: https://docs.microsoft.com/cli/azure
- **Azure Architecture Center**: https://docs.microsoft.com/azure/architecture
- **Microsoft Learn**: https://docs.microsoft.com/learn
- **Azure Friday**: https://azure.microsoft.com/resources/videos/azure-friday
- **Azure Charts**: https://azurecharts.com

---

**Author**: Azure Solutions Architect
**Last Updated**: 2024
**Version**: 1.0

