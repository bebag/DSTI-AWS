# Configuring a JumpBox on AWS (checklist)

- [ ] **Create a VPC**

      - Services -> VPC to get the VPC Dashboard and select "Your VPCs" … Click on "Create VPC"
        - Name tag: JumpBox VPC
        - IPv4 CIDR block: 10.10.0.0/16
      - Yes, Create

- [ ] **Create a public subnet in the VPC**

      - in the VPC Dashboard … select  "Subnets" and click on "Create Subnet"
        - Name tag: JumpBox Public Subnet
        - VPC: select the VPC with the tag "JumpBox VPC"
        - IPv4 CIDR block: 10.10.10.0/24
      - Yes, Create

- [ ] **Create a private subnet in the VPC**

      - in the VPC Dashboard … select  "Subnets" and click on "Create Subnet"
        - Name tag: JumpBox Private Subnet
        - VPC: select the VPC with the tag "JumpBox VPC"
        - IPv4 CIDR block: 10.10.11.0/24
      - Yes, Create

- [ ] **Create an Internet Gateway**

      - in the VPC Dashboard … select  "Internet Gateways" and click on "Create Internet Gateway"
        - Name tag: JumpBox IGW
      - Yes, Create
      - Click on "Attach to VPC" and select the "JumpBox VPC" … Yes, Attach

- [ ] **Create a NAT Gateway in the public subnet**

      - in the VPC Dashboard … select  "NAT Gateways" and click on "Create NAT Gateway"
        - Subnet: select the JumpBox Public Subnet
        - Elastic IP allocation ID: Click on "Create new EIP"
      - Create a NAT Gateway
      - Close

- [ ] **Create/Configure a "public" route table for the public subnet** 

      - in the VPC Dashboard … select  "Route Tables" and click on "Create Route Table"
        - Name tag: Public JumpBox RTB
        - VPC: select the VPC with the tag "JumpBox VPC"
      - Yes, Create
      - Click on "Subnet Associations" … Edit
        - select the JumpBox Public Subnet and click on "Save"
      - Click on "Routes" … Edit
        - Add another route … Destination: 0.0.0.0/0 and select the JumpBOX IGW for the Target
        - "Save"

- [ ] **Create/Configure a "private" route table for the public subnet** 

      - in the VPC Dashboard … select  "Route Tables" and click on "Create Route Table"
        - Name tag: Private JumpBox RTB
        - VPC: select the VPC with the tag "JumpBox VPC"
      - Yes, Create
      - Click on "Subnet Associations" … Edit
        - select the JumpBox Private Subnet and click on "Save"
      - Click on "Routes" … Edit
        - Add another route … Destination: 0.0.0.0/0 and select the nat gateway for the Target
        - "Save"

- [ ] **Create an EC2 instance with a public IP in the public subnet**

      - Services -> EC2 … Click on "Launch Instance"
        - select "Amazon Linux AMI 2017.09.1 (HVM), SSD Volume Type - ami-1a962263"
        - t2.micro (Free tier eligible)
        - Configure instance details:
          - Network: JumpBox VPC
          - Subnet: JumpBox Public Subnet
          - Auto-assign Public IP: Enable
          - Network Interfaces: Primary IP: 10.10.10.10
        - Add Storage (keep the default)
        - Add Tags (Key: NAME and Value: PUB-JB)
        - Configure Security Groups
          - Create a new Security Group
          - Security Group Name: SG-PUB-JB
          - Add rule (on top of the default SSH rule)
            - All ICMP - IPv4 : source = 0.0.0.0/0
        - Review and Launch
        - Launch and either choose an existing pair or create a new one
        - View Instances

- [ ] **Create an EC2 instance with a private IP in the public subnet**

      - Services -> EC2 … Click on "Launch Instance"
        - select "Amazon Linux AMI 2017.09.1 (HVM), SSD Volume Type - ami-1a962263"

        - t2.micro (Free tier eligible)

        - Configure instance details:
          - Network: JumpBox VPC
          - Subnet: JumpBox Private Subnet
          - Auto-assign Public IP: Disable
          - Network Interfaces: Primary IP: 10.10.11.11
          - Advanced Details … User data As text

                   #!/bin/bash
                   printf "\nMatch Address 10.10.0.0/16" >> /etc/ssh/sshd_config
                   printf "\nPasswordAuthentication yes" >> /etc/ssh/sshd_config
                   /etc/init.d/sshd reload
                   useradd jb-user
                   echo jb-user:changeme | chpasswd

        - Add Storage (keep the default)

        - Add Tags (Key: NAME and Value: PRIV-JB)
        - Configure Security Groups
          - Create a new Security Group
          - Security Group Name: SG-PRIV-JB
          - Modify the default rule to only allow ssh from your VPC: Source = 10.10.0.0/16
          - Add rule
            - All ICMP - IPv4 : source = 10.10.0.0/16

        - Review and Launch

        - Launch and select "Proceed without a key pair"

        - View Instances


- [ ] **Test**

      - Connect to PUB-JB:
        - ssh -i <pem file> ec2-user@<your_IP>
      - From PUB-JB, connect to PRIV-JB:
        - ssh jb-user@10.10.11.11 (password:changeme)
      - From PUB-JB, ping a public IP:
        - ping www.amazon.com

