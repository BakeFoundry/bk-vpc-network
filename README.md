# bk-vpc-network

## AWS Network Architecture

This repository contains the configuration for a resilient AWS network with the following architecture:
- **Virtual Private Cloud (VPC)** spanning two Availability Zones (AZs).
- **Subnets**: Each AZ has its own distinct Public and Private subnets.
- **Route Tables**:
  - **Public Route Table**: Associated with public subnets, routing traffic to the Internet Gateway.
  - **Private Route Table**: Associated with private subnets, routing outbound traffic to NAT Gateways.
- **Networking & Connectivity**: 
  - External traffic entry via **Amazon Route 53**.
  - **NAT Gateways** (one per AZ) reside in public subnets to provide secure outbound internet access for private resources.
- **Compute & Load Balancing**: 
  - An **Application Load Balancer (ALB)** in the public subnets handles external traffic distribution.
  - An **Auto Scaling Group (ASG)** in the private subnets manages the lifecycle of the application servers.
- **Security Groups**:
  - **ALB Security Group**: Allows inbound HTTP/HTTPS traffic from the internet and outbound traffic to the Application Security Group.
  - **Application Security Group**: Allows inbound traffic *only* from the ALB Security Group, ensuring the servers are not directly accessible from the internet.

### Network Architecture Diagram

```mermaid
flowchart TB
    User([External User])
    R53((Amazon Route 53))
    IGW[Internet Gateway]

    subgraph AWS [AWS Cloud]
        subgraph VPC [Virtual Private Cloud]
            direction TB
            
            subgraph PublicSubnets [Public Subnets - AZ1 & AZ2]
                direction LR
                ALB[Application Load Balancer]
                NAT1[NAT Gateway 1]
                NAT2[NAT Gateway 2]
            end

            subgraph PrivateSubnets [Private Subnets - AZ1 & AZ2]
                direction TB
                subgraph ASG [Auto Scaling Group]
                    App1[App Server 1]
                    App2[App Server 2]
                end
                PRT[Private Route Table]
            end

            IGW <--> ALB
            ALB --> App1
            ALB --> App2
            
            %% Outbound Traffic Flow
            App1 -.-> PRT
            App2 -.-> PRT
            PRT -.-> NAT1
            PRT -.-> NAT2
            NAT1 -.-> IGW
            NAT2 -.-> IGW
        end
    end

    User <--> R53
    R53 <--> IGW

    %% Styling
    classDef public fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef private fill:#ffebee,stroke:#b71c1c,stroke-width:2px;
    classDef global fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    
    class PublicSubnets public;
    class PrivateSubnets private;
    class R53,AWS,VPC global;
```
