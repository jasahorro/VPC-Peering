<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# VPC Peering

**Project Link:** [View Project](https://nextwork.ai/projects/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_88727bef)

## Introducing Today's Project!

### What is Amazon VPC?

Amazon VPC (Virtual Private Cloud) is a logically isolated virtual network dedicated to your AWS account, and it is useful because it allows you to securely launch, configure, and run AWS resources (like EC2 instances and databases) in a controlled virtual network environment complete with customizable subnets, route tables, and security gateways.

### How I used Amazon VPC in this project

In today's project, I used Amazon VPC to set up multiple isolated virtual networks, establish a direct VPC peering connection between them, update route tables for cross-network communication, launch EC2 instances into public subnets, assign static Elastic IPs for browser-based connectivity, and configure security group rules to test and validate private network routing using ping diagnostic commands.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how default security groups completely block all inbound traffic—including ICMP ping tests between peered VPCs—requiring explicit manual rules just to allow basic cross-network communication.

### This project took me...

This project took me 40 minutes only.

## In the first part of my project...

### Step 1 - Set up my VPC

In this step, I will create two separate Amazon VPCs with public subnets and unique IPv4 CIDR blocks using the VPC and more wizard because it allows us to set up the foundational cloud networking architecture required to establish a VPC peering connection between them.

### Step 2 - Create a Peering Connection

In this step, I will create a VPC peering connection between NextWork-1-vpc (Requester) and NextWork-2-vpc (Accepter) because it establishes a direct, private communication link between the two virtual private clouds using their private IP addresses without routing through the public internet.

### Step 3 - Update Route Tables

In this step, I will add a route to NextWork-1-vpc's route table pointing to NextWork-2-vpc's CIDR block through the peering connection because it enables traffic from VPC 1 to successfully route to VPC 2.

### Step 4 - Launch EC2 Instances

I will launch an EC2 instance in each VPC because they provide the servers needed to test network communication and verify our VPC peering connection.

## Multi-VPC Architecture

I started my project by launching two Amazon VPCs (NextWork-1-vpc with CIDR 10.1.0.0/16 and NextWork-2-vpc with CIDR 10.2.0.0/16) using the VPC and more wizard, each configured with 1 public subnet and 0 private subnets.

The CIDR blocks for VPCs 1 and 2 are 10.1.0.0/16 and 10.2.0.0/16. They have to be unique because overlapping IP address ranges across peered VPCs cause routing conflicts, preventing resources from determining whether traffic should stay local or route through the peering connection.

### I also launched 2 EC2 instances

I didn't set up key pairs for these EC2 instances because I connected with EC2 Instance Connect, which pushes a temporary SSH key for each session. This means I can securely access my instances without manually downloading and managing .pem key files.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_11111111)

## VPC Peering

A VPC peering connection is a direct network route between two Amazon Virtual Private Clouds that enables resources within them to communicate securely using private IPv4 addresses without traversing the public internet.

VPCs would use peering connections when resources in separate networks need to talk to each other privately, for example an app in one VPC accessing a database in another, or sharing resources between different AWS accounts or teams.

The difference between a Requester and an Accepter in a peering connection is that the Requester is the VPC that initiates and sends the connection invitation, while the Accepter is the VPC that receives the request and must approve it before the peering link becomes active.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_1cbb1b88)

## Updating route tables

After accepting a peering connection, my VPCs' route tables need to be updated because traffic in each VPC needs to know where to send packets destined for the other VPC's network range, as traffic won't automatically route through the peering connection without an explicit route entry pointing to it.

My VPCs' new routes have a destination of the peered VPC's CIDR block (10.2.0.0/16 for VPC 1 and 10.1.0.0/16 for VPC 2). The routes' target was the VPC peering connection (pcx-0e380f223cec0713d).

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_4a9e8014)

## In the second part of my project...

### Step 5 - Use EC2 Instance Connect

In this step, I will use EC2 Instance Connect to securely connect to my first EC2 instance (Instance - NextWork VPC 1) via a browser-based SSH terminal because it allows us to test network connectivity and troubleshoot communication with our peered VPC.

### Step 6 - Connect to EC2 Instance 1

In this step, I will troubleshoot and update the security group inbound rules for Instance - NextWork VPC 1 because the default security group blocks incoming traffic from outside the VPC, and we need to allow inbound SSH traffic on port 22 from the public internet to successfully connect via EC2 Instance Connect.

### Step 7 - Test VPC Peering

In this step, I will test the VPC peering connection by sending a ping command from the first EC2 instance to the private IPv4 address of the second EC2 instance because it verifies whether the two peered virtual private clouds can successfully communicate with each other over the private network.

## Troubleshooting Instance Connect

Next, I tried to connect to Instance - NextWork VPC 1 using EC2 Instance Connect so I could run a ping test from inside it, but the connection failed.

I was stopped from using EC2 Instance Connect because my instance lacked a public IPv4 address, as public IP assignment was disabled during creation, preventing connection over the public internet.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_7685490c)

## Elastic IP addresses

To resolve this error, I set up Elastic IP addresses. Elastic IP addresses are static IPv4 addresses that get allocated to your AWS account, giving your EC2 instances a permanent public IP address that doesn't change when they are restarted.

Associating an Elastic IP address resolved the error because it provides the instance with a static public IP address, allowing EC2 Instance Connect to securely reach and establish an SSH terminal session over the public internet.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_45663498)

## Troubleshooting ping issues

To test VPC peering, I ran the command ping 10.2.0.0

A successful ping test validates your VPC peering connection because it confirms that two separate Virtual Private Clouds can successfully route, transmit, and receive network traffic using their private IP addresses across the established peering link without relying on the public internet.

I had to update my second EC2 instance's security group because the default security group blocks all incoming traffic from outside the VPC. I added a new rule that allows All ICMP - IPv4 traffic from the first VPC's CIDR block (10.1.0.0/16) as the source, enabling ping test packets to successfully reach the instance across the peering connection.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16_7a29d352)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/fd75209e-d2a1-5bdd-90e8-bdb9ee49de16)*
