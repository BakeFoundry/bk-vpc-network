# bk-vpc-network

## AWS Network Architecture

This repository contains the configuration for a resilient AWS network with the following architecture:
- **Virtual Private Cloud (VPC)** spanning two Availability Zones (AZs).
- **Subnets**: Each AZ has its own distinct Public and Private subnets.
- **Routing & Networking**: 
  - External traffic comes through **Amazon Route 53**.
  - **NAT Gateways** in each AZ's Public Subnet provide outbound internet access for the application servers in the Private Subnets.
- **Compute & Load Balancing**: 
  - An **Application Load Balancer (ALB)** is placed in the Public Subnets to distribute incoming external traffic.
  - An **Auto Scaling Group (ASG)** spans the Private Subnets, provisioning the application servers where the app is deployed.

### Network Diagram

![AWS Network Architecture](network-architecture.svg)