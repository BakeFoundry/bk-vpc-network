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
flowchart TD
    %% Define AWS Icons for diagram elements
    classDef awsIcon fill:none,stroke:none,color:#000
    
    %% Users
    User["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/General/User.png' width='50' height='50' /><br>External Traffic"]:::awsIcon
    
    %% AWS Services
    R53["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/Route53.png' width='50' height='50' /><br>Amazon Route 53"]:::awsIcon
    
    ALB["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/ElasticLoadBalancing.png' width='50' height='50' /><br>Load Balancer"]:::awsIcon
    
    ALB1["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/ElasticLoadBalancing.png' width='50' height='50' /><br>ALB Node 1"]:::awsIcon
    ALB2["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/ElasticLoadBalancing.png' width='50' height='50' /><br>ALB Node 2"]:::awsIcon
    
    NAT1["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/NATGateway.png' width='50' height='50' /><br>NAT Gateway 1"]:::awsIcon
    NAT2["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/NetworkingAndContentDelivery/NATGateway.png' width='50' height='50' /><br>NAT Gateway 2"]:::awsIcon
    
    App1["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/Compute/EC2Instance.png' width='50' height='50' /><br>App Server Instances"]:::awsIcon
    App2["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/Compute/EC2Instance.png' width='50' height='50' /><br>App Server Instances"]:::awsIcon
    
    ASG["<img src='https://raw.githubusercontent.com/awslabs/aws-icons-for-plantuml/main/dist/Compute/AutoScaling.png' width='50' height='50' /><br>Auto Scaling Group"]:::awsIcon
    
    User --> R53
    
    subgraph AWS [AWS Cloud]
        subgraph VPC [Virtual Private Cloud]
            %% Route 53 points to ALB
            R53 --> ALB
            
            subgraph AZ1 [Availability Zone 1]
                direction TB
                subgraph Public1 [Public Subnet]
                    ALB1
                    NAT1
                end
                subgraph Private1 [Private Subnet]
                    App1
                end
            end
            
            subgraph AZ2 [Availability Zone 2]
                direction TB
                subgraph Public2 [Public Subnet]
                    ALB2
                    NAT2
                end
                subgraph Private2 [Private Subnet]
                    App2
                end
            end
            
            %% ALB distributes traffic
            ALB --> ALB1
            ALB --> ALB2
            ALB1 --> App1
            ALB2 --> App2
            
            %% App servers connect to NAT for outbound
            App1 -.->|Outbound| NAT1
            App2 -.->|Outbound| NAT2
            
            %% Auto scaling controls app servers
            ASG -.- App1
            ASG -.- App2
        end
    end
    
    NAT1 -.->|To Internet| User
    NAT2 -.->|To Internet| User

    %% Styling for Subnets
    classDef publicSubnet fill:#e6f3ff,stroke:#3388ff,stroke-width:2px,color:#000,stroke-dasharray: 5 5;
    classDef privateSubnet fill:#ffe6e6,stroke:#ff3333,stroke-width:2px,color:#000,stroke-dasharray: 5 5;
    class Public1,Public2 publicSubnet;
    class Private1,Private2 privateSubnet;
    classDef cloudBox fill:none,stroke:#232f3e,stroke-width:2px,color:#000;
    class AWS,VPC,AZ1,AZ2 cloudBox;
```