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

```mermaid
flowchart TB
    User([External Traffic])
    R53((Amazon Route 53))
    
    User --> R53

    subgraph AWS [AWS Cloud]
        subgraph VPC [Virtual Private Cloud]
            R53 --> ALB{{Load Balancer}}
            
            subgraph AZ1 [Availability Zone 1]
                direction TB
                subgraph Public1 [Public Subnet 1]
                    ALB1[ALB Node 1]
                    NAT1(NAT Gateway 1)
                end
                subgraph Private1 [Private Subnet 1]
                    App1[App Server Instances]
                end
            end
            
            subgraph AZ2 [Availability Zone 2]
                direction TB
                subgraph Public2 [Public Subnet 2]
                    ALB2[ALB Node 2]
                    NAT2(NAT Gateway 2)
                end
                subgraph Private2 [Private Subnet 2]
                    App2[App Server Instances]
                end
            end
            
            ALB --> ALB1
            ALB --> ALB2
            ALB1 --> App1
            ALB2 --> App2
            App1 -.->|Outbound| NAT1
            App2 -.->|Outbound| NAT2
            
            ASG[Auto Scaling Group] -.- App1
            ASG -.- App2
        end
    end
    
    NAT1 -.->|To Internet| User
    NAT2 -.->|To Internet| User

    classDef publicSubnet fill:#e6f3ff,stroke:#3388ff,stroke-width:2px,color:#000;
    classDef privateSubnet fill:#ffe6e6,stroke:#ff3333,stroke-width:2px,color:#000;
    class Public1,Public2 publicSubnet;
    class Private1,Private2 privateSubnet;
```