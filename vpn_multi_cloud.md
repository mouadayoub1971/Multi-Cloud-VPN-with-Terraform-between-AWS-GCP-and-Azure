# Tutorial: Creating a Multi-Cloud VPN with Terraform between AWS, GCP, and Azure

## Overview

Silectis demonstrates how to establish private network connections across AWS, Google Cloud, and Azure using site-to-site VPN connections managed through Terraform infrastructure-as-code.

## Key Purpose

The company needed to connect Magpie clusters deployed across multiple cloud providers to privately access internal infrastructure resources that reside exclusively on AWS.

## Infrastructure Architecture

The design uses AWS as a central hub with these components:

- **AWS hub VPC** hosting shared private resources
- **AWS Simple AD** service for DNS resolution across clouds
- **Site-to-site VPN connections** linking Google Cloud and Azure to the AWS hub
- **Dual VPN tunnels** on each connection for high availability
- **VPC peering** for connecting other AWS VPCs to the hub

## Implementation Approach

The tutorial walks through a Terraform project structured with:

- Provider configurations for AWS, Google Cloud, and Azure
- Separate modules for AWS directory services, virtual networks, and VPN infrastructure
- Declarative infrastructure code allowing version control

## Configuration Steps

**AWS setup** includes creating a VPC with public/private subnets and configuring Simple AD for DNS services.

**Google Cloud** requires a private VNet with Cloud DNS conditional forwarding to Simple AD, enabling instances to resolve internal AWS hostnames.

**Azure** Virtual Networks directly use Simple AD servers as DNS, since Azure Private DNS doesn't natively support the necessary forwarding.

## VPN Connection Details

The approach uses BGP routing for Google Cloud connections but requires manual route configuration for Azure due to compatibility limitations between AWS and Azure BGP implementations.

VPN activation typically requires 10-20 minutes post-deployment.

## Author

Jon Lounsbury, VP of Engineering at Silectis, with presence on GitHub and LinkedIn.

---

*Source: https://www.silect.is/blog/multi-cloud-vpn-terraform/*
