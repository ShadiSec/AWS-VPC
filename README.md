# AWS Virtual Private Cloud (VPC)

**Table of Contents**
1. [Introduction](#Overview)
2. [Definitions](#Definitions)
3. [Setup](#Set-Up)
   - [Creating a VPC](#Creating-a-VPC)
   - [NAT Gateway](#NAT-Gateway)
   - [Creating a Bastion Host](#Creating-the-Bastion-Host)
   - [Creating Private Instance](#Creating-the-Private-Instance)
   - [Connecting to the Private Instance](#Connecting-to-the-Private-Instance)
     - [Loading SSH Key](#-Loading-your-SSH-Private-Key)
     - [Connecting to Bastion](#-Connecting-to-the-Bastion)
     - [Hopping to the Private Instance](#Hopping-to-the-Private-Instance)

# Overview

In this write-up, I will run you through the process of setting up:

- Virtual Private Cloud within AWS with a Public and Private Subnet
- Create and use a Bastion Host
- Set up Route Rules
- Use SSH Agent Forwarding

# Definitions

1. **Virtual Private Cloud (VPC):** A logically isolated network that separates your instances from the rest of the AWS clients.
2. **Bastion Host:** An intermediate host (Jump Box), sitting on the public subnet, allowing you to connect to the main host that is hidden behind a private subnet, which is only accessible by the bastion host.
3. **SSH Agent Forwarding:** A method that allows the user to forward their SSH Private Key from their local device to another host without storing it on the host at the next hop. 

# Set Up

## Creating a VPC

1. In the AWS Portal, search for the VPC Service.
2. Once in the VPC dashboard, click on “Create VPC.”
3. In the VPC Wizard:
    1. Resources to create = VPC and more (this allows us to create the subnets)
    2. IPv4 CIDR Block = Give your VPC a CIDR to determine its size (e.g., 172.16.8.0/22).
    3. Number of Public Subnets = 1
    4. Number of Private Subnets = 1
    5. Customize Subnets CIDR Blocks:
        1. Public = 172.16.8.0/26 (64 IPs)
        2. Private = 172.16.9.0/23 (512 IPs)
    6. Leave everything else as the default and click “Create VPC.”

## NAT Gateway

The NAT Gateway allows our instances to communicate with the internet by routing their traffic through the Internet Gateway (Public IP). By default, the VPC wizard does this for us when we select “VPC and more” in the setup wizard.

- The Public Subnet's Route Table is routed to the Internet Gateway (IGW).
- The Private Subnet's Route Table is routed to the NAT Gateway.

## Creating the Bastion Host

1. Navigate to the EC2 service dashboard.
2. Click “Launch Instance.”
3. In the wizard:
    1. OS Image = Amazon Linux
    2. Key Pair = Select your key pair:
        1. Click “create” if not available and select:
            1. “.pem” if using a Linux machine 
            2. “.pkk” if using Putty
    3. Network Settings:
        1. Click the “Edit” Button
        2. VPC = Select the VPC we created earlier
        3. Subnet = Select the Public subnet
        
        ![image.png](/images/image.png)
        
        1. Security group name = Name it something like BastionSG
        2. Firewall:
            1. Allow SSH Traffic from “My IP.”
            
            ![image.png](/images/image-1.png)
            
4. Leave everything else as the default and click “Launch Instance.”

## Creating the Private Instance

1. In the EC2 dashboard.
2. Click “Launch Instance.”
3. In the wizard:
    1. OS Image = Ubuntu
    2. Key Pair = Select your key pair (Should be the same one you selected for the Bastion Host instance creation).
    3. Network Settings:
        1. Click the “Edit” Button
        2. VPC = Select the VPC we created earlier
        3. Subnet = Select the Private subnet
        
        ![image.png](/images/image-2.png)
        
        1. Security group name = Name it something like PrivateLinuxSG.
        2. Firewall:
            1. Allow SSH Traffic from the “Custom” Source type
            2. In the “Source”, select the bastion security group created earlier.
            
            ![image.png](/images/image-3.png)
            
4. Leave everything else as the default and click “Launch Instance.”

## Connecting to the Private Instance

### Loading your SSH Private Key

1. Locate your SSH key on your machine.
2. Open a terminal and run:
    
    ```bash
    ssh-add /path/to/ssh_key.pem
    ```
    
    1. If you get a warning about “Unprotected Permissions,” run:
        
        ```bash
        # This sets the file permissions
        # to only be readable by the owner (you).
        chmod 400 /path/to/ssh_key.pem
        ```
        
    2. Then run the ssh-add command again
3. To check if the key was added, run:
    
    ```bash
    ssh-add -l
    ```
    

### Connecting to the Bastion

1. Go to your EC2 Dashboard and click on “Instances (running).”
2. Grab your Bastion’s Public IP.
3. Open your terminal and run:
    
    ```bash
    # The default user for Amazon Linux is 'ec2-user'.
    # The '-A' enables SSH Agent Forwarding for the current connection.
    ssh -A <user>@<bastion-IP>
    ```
    
4. If everything was successful, you’re now in the Bastion and ready to hop to the Private Instance.

### Hopping to the Private Instance

1. Navigate to your EC2 dashboard and grab the Private IP of your Private Instance. 
2. In the Bastion Terminal run:
    
    ```bash
    # Default user for Ubuntu is ubuntu
    ssh <user>@<Private-Instance-IP>
    ```
    
3. If everything was successful, you should now be inside the Private Instance. 
    
    ![image.png](/images/image-4.png)
