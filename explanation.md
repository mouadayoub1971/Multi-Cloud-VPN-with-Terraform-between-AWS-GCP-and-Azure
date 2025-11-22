# Multi-Cloud VPN Infrastructure: Detailed Explanation

## Overview

This document provides a comprehensive explanation of creating a multi-cloud VPN infrastructure that connects AWS, Google Cloud Platform (GCP), and Microsoft Azure using Terraform. The architecture enables private communication between resources across all three cloud providers.

## Table of Contents

1. [Business Problem](#business-problem)
2. [Architecture Overview](#architecture-overview)
3. [Network Design](#network-design)
4. [Technology Stack](#technology-stack)
5. [Component Breakdown](#component-breakdown)
6. [Implementation Details](#implementation-details)
7. [Deployment Process](#deployment-process)

---

## Business Problem

**Challenge**: Silectis deploys their Magpie data engineering clusters across AWS, Google Cloud, and Azure. However, some critical internal infrastructure resides exclusively on AWS. Resources deployed on GCP and Azure need secure, private access to these AWS-hosted services.

**Solution**: Establish a hub-and-spoke network architecture using site-to-site VPN connections that enable:
- Private connectivity between all three cloud providers
- Secure communication without traversing the public internet
- Centralized DNS resolution for private hostnames
- High availability through redundant VPN tunnels

---

## Architecture Overview

### Hub-and-Spoke Topology

```
         ┌─────────────┐
         │   AWS VPC   │
         │    (Hub)    │
         │             │
         │  - VPN GW   │
         │  - DNS (AD) │
         └──────┬──────┘
                │
        ┌───────┴───────┐
        │               │
   ┌────▼────┐     ┌───▼─────┐
   │  GCP    │     │  Azure  │
   │  VNet   │     │  VNet   │
   │ (Spoke) │     │ (Spoke) │
   └─────────┘     └─────────┘
```

**Key Design Principles**:
- AWS serves as the central hub
- GCP and Azure connect to AWS via site-to-site VPNs
- Each VPN connection has 2 tunnels for high availability
- DNS is centralized in AWS using Simple AD

---

## Network Design

### IP Address Allocation

| Cloud Provider | Network Name | CIDR Block | Subnet Type |
|---------------|--------------|------------|-------------|
| AWS | aws-test | 10.0.0.0/16 | Public & Private |
| GCP | google-test | 10.1.0.0/16 | Private only |
| Azure | azure-test | 10.2.0.0/16 | Public & Private |

### Subnet Breakdown

#### AWS Subnets
- **Private Subnets**: `cidrsubnet(10.0.0.0/16, 4, count.index * 2 + 1)`
- **Public Subnets**: `cidrsubnet(10.0.0.0/16, 4, count.index * 2 + 2)`
- **DNS Subnets**: `10.0.0.0/20` (for Simple AD service)

#### GCP Subnets
- **Private Subnets**: `cidrsubnet(10.1.0.0/16, 4, count.index)`
- No public subnets (GCP uses Cloud NAT for outbound internet access)

#### Azure Subnets
- **Private Subnets**: `cidrsubnet(10.2.0.0/16, 4, count.index * 2 + 2)`
- **Public Subnets**: `cidrsubnet(10.2.0.0/16, 4, count.index * 2 + 1)`
- **Gateway Subnet**: `10.2.0.0/27` (required for VPN gateway)

---

## Technology Stack

### Infrastructure as Code
- **Terraform**: Version-controlled, declarative infrastructure management
- **Provider Versions**:
  - AWS: ~> 2.39.0
  - Google: ~> 2.18.0
  - Google Beta: ~> 2.18.0 (for DNS private forwarding)
  - Azure: ~> 1.41.0

### Authentication Methods
- **AWS**: Shared credentials file or environment variables
- **GCP**: Service account key file (`~/.config/gcloud/${project_id}.json`)
- **Azure**: CLI authentication

---

## Component Breakdown

### 1. AWS Infrastructure

#### VPC Components
```terraform
aws_vpc
├── Public Subnets
├── Private Subnets
├── Internet Gateway (IGW)
├── NAT Gateway
├── Public Route Table (routes to IGW)
└── Private Route Table (routes to NAT)
```

#### Network ACLs (NACLs)

**Public Subnet NACL Rules**:
- Allow HTTP (port 80) inbound
- Allow HTTPS (port 443) inbound
- Allow ephemeral ports (1024-65535) TCP/UDP inbound
- Allow all traffic from VPC CIDR
- Allow all outbound traffic

**Private Subnet NACL Rules**:
- Allow ephemeral ports (1024-65535) TCP/UDP inbound
- Allow all traffic from VPC CIDR
- Allow all outbound traffic

#### DNS Service (AWS Simple AD)
```terraform
aws_directory_service_directory
├── Type: SimpleAD
├── Size: Small
├── Dedicated subnets in 2 availability zones
├── Network ACL for DNS traffic
└── Provides DNS forwarding to other clouds
```

**Purpose**: Centralized DNS service that allows GCP and Azure resources to resolve AWS private hostnames.

**DNS Network ACL**: Special rules to allow DNS traffic (TCP/UDP port 53) from:
- GCP subnet CIDR ranges
- Azure subnet CIDR ranges

#### VPN Gateway
```terraform
aws_vpn_gateway
├── Attached to VPC
├── Route propagation enabled
├── Customer Gateway for GCP (BGP-enabled)
├── Customer Gateway for Azure (static routes)
├── VPN Connection to GCP (2 tunnels, BGP)
└── VPN Connection to Azure (2 tunnels, static)
```

### 2. Google Cloud Infrastructure

#### Network Components
```terraform
google_compute_network (GLOBAL routing)
├── Private Subnets
├── Cloud Router (for NAT)
└── Cloud NAT (for internet access)
```

**Note**: GCP doesn't require public subnets due to its load balancing architecture.

#### Cloud DNS Configuration
```terraform
google_dns_managed_zone
├── Visibility: Private
├── Attached to VPC network
└── Conditional forwarding to AWS Simple AD
```

**Purpose**: Allows GCP instances to resolve AWS private DNS names by forwarding queries to Simple AD.

#### VPN Components
```terraform
google_compute_vpn_gateway
├── Static IP address
├── Forwarding rules (ESP, UDP 500, UDP 4500)
├── VPN Tunnel 1
│   ├── Cloud Router 1
│   ├── BGP Peer 1
│   └── Router Interface 1
└── VPN Tunnel 2
    ├── Cloud Router 2
    ├── BGP Peer 2
    └── Router Interface 2
```

**BGP Configuration**:
- ASN: 65000
- Advertise mode: CUSTOM
- Advertised groups: ALL_SUBNETS
- Additional advertised ranges for external DNS

### 3. Azure Infrastructure

#### Virtual Network Components
```terraform
azurerm_virtual_network
├── DNS servers: AWS Simple AD IPs
├── Private Subnets
├── Public Subnets
└── Gateway Subnet (named "GatewaySubnet" - required by Azure)
```

**Note**: Azure Virtual Network directly uses Simple AD servers for DNS (no conditional forwarding like GCP).

#### VPN Components
```terraform
azurerm_virtual_network_gateway
├── Type: VPN
├── VPN Type: RouteBased
├── BGP: Disabled (incompatible with AWS BGP)
├── Public IP address
├── Local Network Gateway 1 (points to AWS tunnel 1)
│   └── Connection 1
└── Local Network Gateway 2 (points to AWS tunnel 2)
    └── Connection 2
```

**Important**: Azure and AWS have BGP incompatibility issues, so static routing is used instead.

---

## Implementation Details

### VPN Tunnel Configuration

#### AWS ↔ GCP VPN
- **Protocol**: IPsec (ipsec.1)
- **Routing**: BGP (Border Gateway Protocol)
- **Tunnels**: 2 for high availability
- **IKE Version**: 1
- **Shared Secret**: Auto-generated by AWS, consumed by GCP

**BGP Details**:
```
AWS Side:
- BGP ASN: Auto-assigned by AWS
- Inside IP addresses: Auto-assigned

GCP Side:
- BGP ASN: 65000
- Inside IP addresses: Match AWS tunnel configuration
- Router peers configured for each tunnel
```

#### AWS ↔ Azure VPN
- **Protocol**: IPsec (ipsec.1)
- **Routing**: Static routes
- **Tunnels**: 2 for high availability
- **Shared Secret**: Auto-generated by AWS, consumed by Azure

**Static Routes**:
- AWS maintains routes to Azure network CIDR blocks
- Configured via `aws_vpn_connection_route` resources

### DNS Resolution Flow

#### GCP → AWS
1. GCP instance queries private AWS hostname
2. Query sent to Cloud DNS
3. Cloud DNS conditionally forwards to Simple AD
4. Simple AD resolves and returns IP
5. Response flows back through VPN tunnel

#### Azure → AWS
1. Azure VM queries private AWS hostname
2. Query sent directly to Simple AD (configured as VNet DNS)
3. Simple AD resolves and returns IP
4. Response flows back through VPN tunnel

#### Limitations
- AWS instances cannot resolve GCP hostnames (one-way DNS)
- AWS instances cannot resolve Azure hostnames (one-way DNS)
- Solution: GCP supports inbound DNS forwarding if bidirectional DNS is needed

### Network ACL Security

#### DNS Traffic ACL Rules

**For GCP (per subnet)**:
```
Ingress TCP DNS: Rule 200+index, port 53
Ingress UDP DNS: Rule 250+index, port 53
External DNS TCP: Rule 300, port 53
External DNS UDP: Rule 301, port 53
```

**For Azure**:
```
Ingress TCP DNS: Rule 400+index, port 53
Ingress UDP DNS: Rule 450+index, port 53
```

---

## Deployment Process

### 1. Prerequisites
- AWS account with appropriate IAM permissions
- GCP project with Compute API enabled
- Azure subscription with Resource Group
- Terraform installed locally
- Cloud provider credentials configured

### 2. Variable Configuration

Create a `terraform.tfvars` file with:
```hcl
# AWS
aws_region                    = "us-east-1"
aws_directory_service_password = "SecurePassword123!"
aws_dns_suffix                = "ec2.internal"

# GCP
google_project_id = "my-gcp-project"
google_region     = "us-central1"

# Azure
azure_resource_group_name = "my-resource-group"
azure_location           = "eastus"
```

### 3. Terraform Commands

```bash
# Initialize Terraform and download providers
terraform init

# Review the planned infrastructure changes
terraform plan

# Apply the configuration
terraform apply

# Confirm with: yes
```

### 4. Wait Time
- VPN connections typically take 10-20 minutes to become active after Terraform completes
- Monitor VPN tunnel status in each cloud provider's console

### 5. Testing Connectivity

**Test Setup**:
1. Deploy an EC2 instance in AWS private subnet
2. Configure a web server on port 1024+ (to pass NACL rules)
3. Deploy a GCP Compute instance in private subnet
4. Deploy an Azure VM in private subnet

**Test Commands**:
```bash
# From GCP instance
curl http://<aws-private-ip>:8080

# From Azure VM
curl http://<aws-private-ip>:8080

# Test DNS resolution from GCP
nslookup <aws-private-hostname>

# Test DNS resolution from Azure
nslookup <aws-private-hostname>
```

---

## Module Structure

### Project Organization
```
multi-cloud-vpn/
├── main.tf                    # Main entry point
├── variables.tf               # Variable declarations
├── providers.tf               # Provider configurations
├── aws/
│   ├── vpc/
│   │   └── main.tf           # AWS VPC resources
│   └── dns/
│       └── main.tf           # Simple AD resources
├── google/
│   └── network/
│       └── main.tf           # GCP network resources
├── azure/
│   └── vnet/
│       └── main.tf           # Azure VNet resources
└── vpn/
    ├── aws.tf                # AWS VPN configuration
    ├── google.tf             # GCP VPN configuration
    └── azure.tf              # Azure VPN configuration
```

### Module Instantiation

```hcl
# Main configuration brings everything together

module "aws_vpc" {
  source       = "./aws/vpc"
  vpc_name     = "aws-test"
  cidr_block   = "10.0.0.0/16"
  subnet_count = 1
}

module "dns" {
  source                 = "./aws/dns"
  vpc_id                 = module.aws_vpc.vpc_id
  directory_name         = "test.internal"
  directory_password     = var.aws_directory_service_password
  dns_subnet_cidr_prefix = "10.0.0.0/20"
  private_route_table_id = module.aws_vpc.private_route_table_id
}

module "google_network" {
  source               = "./google/network"
  project_id           = var.google_project_id
  network_name         = "google-test"
  cidr_block           = "10.1.0.0/16"
  regions              = [var.google_region]
  aws_dns_suffix       = var.aws_dns_suffix
  aws_dns_ip_addresses = module.dns.dns_ip_addresses
}

module "azure_vnet" {
  source              = "./azure/vnet"
  resource_group_name = var.azure_resource_group_name
  location            = var.azure_location
  network_name        = "azure-test"
  cidr_block          = "10.2.0.0/16"
  subnet_count        = 1
  dns_servers         = module.dns.dns_ip_addresses
}

module "vpn" {
  source                      = "./vpn"
  aws_vpc_id                  = module.aws_vpc.vpc_id
  aws_route_table_ids         = [module.aws_vpc.private_route_table_id, module.aws_vpc.public_route_table_id]
  google_project_id           = var.google_project_id
  google_region               = var.google_region
  google_network_name         = module.google_network.network_name
  google_subnet_self_links    = module.google_network.private_subnet_self_links
  azure_resource_group_name   = var.azure_resource_group_name
  azure_location              = var.azure_location
  azure_network_name          = module.azure_vnet.network_name
  azure_network_address_space = module.azure_vnet.address_space
  azure_gateway_cidr          = "10.2.0.0/27"
  dns_network_acl_id          = module.dns.dns_network_acl_id
}
```

---

## Key Takeaways

### Benefits of This Architecture

1. **Multi-Cloud Flexibility**: Deploy workloads on the best cloud for each use case
2. **Private Connectivity**: Secure communication without public internet exposure
3. **Centralized DNS**: Single source of truth for private DNS resolution
4. **High Availability**: Dual VPN tunnels prevent single points of failure
5. **Infrastructure as Code**: Reproducible, version-controlled infrastructure
6. **Cost Optimization**: VPN is cheaper than dedicated interconnects for moderate traffic

### Technical Highlights

1. **BGP vs Static Routing**:
   - GCP: BGP enables automatic route propagation
   - Azure: Static routes due to BGP incompatibility

2. **DNS Strategies**:
   - GCP: Conditional forwarding (flexible, allows bidirectional)
   - Azure: Direct DNS server configuration (simpler but less flexible)

3. **Network Security**:
   - Network ACLs control traffic at subnet level
   - VPN encryption secures data in transit
   - Private subnets isolate sensitive resources

4. **Cloud-Specific Considerations**:
   - AWS: Requires separate public/private subnets with NAT
   - GCP: Uses Cloud NAT, no public subnet needed
   - Azure: Gateway subnet must be named "GatewaySubnet"

### Limitations and Considerations

1. **DNS is One-Way**: AWS cannot resolve GCP/Azure hostnames by default
2. **BGP Incompatibility**: AWS and Azure BGP configurations don't align
3. **Latency**: VPN adds overhead compared to dedicated connections
4. **Bandwidth**: VPN throughput limits may impact high-volume data transfers
5. **Cost**: VPN gateway and data transfer charges apply
6. **Complexity**: Multi-cloud introduces operational overhead

---

## Use Cases

This architecture is ideal for:

- **Hybrid Data Platforms**: Deploy data processing on multiple clouds while accessing centralized data stores
- **Multi-Cloud Redundancy**: Distribute workloads across clouds for disaster recovery
- **Cloud Migration**: Gradual migration with maintained connectivity to legacy systems
- **Vendor Flexibility**: Avoid vendor lock-in by maintaining portability
- **Cost Optimization**: Leverage pricing differences across cloud providers
- **Compliance**: Meet data residency requirements while maintaining connectivity

---

## Future Enhancements

Possible improvements to this architecture:

1. **Bidirectional DNS**: Configure inbound DNS forwarding for AWS ↔ GCP/Azure
2. **Transit Gateway**: Use AWS Transit Gateway for scaling to more VPCs
3. **Monitoring**: Add VPN tunnel health monitoring and alerting
4. **Dedicated Connections**: Upgrade to Direct Connect/ExpressRoute/Interconnect for production
5. **Security**: Implement Web Application Firewall (WAF) and DDoS protection
6. **Automation**: Add CI/CD pipelines for infrastructure changes
7. **Mesh Networking**: Enable direct GCP ↔ Azure connectivity if needed

---

## Conclusion

This multi-cloud VPN architecture demonstrates how Terraform can orchestrate complex infrastructure across AWS, GCP, and Azure. By using a hub-and-spoke topology with AWS as the central hub, organizations can achieve private connectivity, centralized DNS, and high availability across multiple cloud providers.

The implementation showcases cloud-agnostic infrastructure-as-code practices, handling provider-specific quirks (like BGP incompatibility and DNS forwarding differences) while maintaining a clean, modular Terraform structure.

For organizations like Silectis deploying data engineering platforms across clouds, this architecture provides the foundation for secure, scalable, multi-cloud operations.

---

**Author**: Jon Lounsbury, VP of Engineering at Silectis
**Source**: Silectis Blog - Tutorial: Creating a Multi-Cloud VPN with Terraform between AWS, GCP, and Azure
**Copyright**: © 2023 Silectis, Inc.
