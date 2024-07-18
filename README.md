# AWS EFS Setup

## Project Overview

This project involves setting up an AWS Elastic File System (EFS) in a VPC with two public subnets and two private subnets. The EFS will be mounted on two EC2 instances located in different public subnets within the same VPC, demonstrating shared storage between multiple EC2 instances.

## Pre-requisites

- An existing VPC named `project-vpc` with 2 public subnets and 2 private subnets.
- AWS CLI configured on your local machine.
- Basic understanding of AWS services and networking.

![EFS Mount Diagram](https://github.com/user-attachments/assets/ea47cb15-f56c-46b4-95d1-16dec27406a3)

## Steps

### Step 1: Create a Security Group

1. **Create a Security Group (nfs-sg):**
    - Name: `nfs-sg`
    - VPC: `project-vpc`
2. **Configure Inbound Rules:**
    - Open all traffic:
        - Type: All Traffic, Protocol: All, Port Range: All, Source: Anywhere (0.0.0.0/0)
    - Open port for NFS (2049):
        - **Type**: Custom TCP, **Protocol**: TCP, **Port Range**: 2049, **Source**: Custom, reference the same security group (`nfs-sg`)
    - Open SSH port (22):
        - Type: SSH, Protocol: TCP, Port Range: 22, Source: Anywhere (0.0.0.0/0)

### Step 2: Launch EC2 Instances

1. **Launch EC2 Instances:**
    - Configure the instances to be within the public subnets of your `project-vpc`.
    - Attach the `nfs-sg` security group to both instances.
    - Ensure each instance is placed in a separate public subnet.
    - Add key pair for SSH access.

### Step 3: Create an Elastic File System (EFS)

1. **Create EFS:**
    - Select the VPC: `project-vpc`
    - Choose the same region as the VPC.
    - Note the EFS ID for later use.

### Step 4: Configure Security Group for EFS

1. **Modify EFS Security Group:**
    - On the EFS Dashboard, select the created EFS.
    - Click on the "Network" tab.
    - Modify the mount target security groups to reference the `nfs-sg` instead of the default security group.

### Step 5: Mount EFS on EC2 Instances

1. **SSH into each EC2 instance:**
    - Use your terminal or SSH client to connect to each EC2 instance using its public IP address.
2. **Install EFS Utilities and Create a Mount Point:**
    ```sh
    sudo yum install -y amazon-efs-utils -y
    sudo mkdir ./efs-mount
    ```
3. **Mount the EFS:**
    ```sh
    sudo mount -t efs -o tls <elastic-file-system-id>:/ ./efs-mount
    ```

### Step 6: Test the EFS

1. **Create a File and Directory:**
    - On the first EC2 instance:
        ```sh
        cd ./efs-mount
        touch sample-file.txt
        mkdir test-directory
        ```
    - On the second EC2 instance, check for the created file and directory:
        ```sh
        cd ./efs-mount
        ls
        ```
2. **Unmount the EFS:**
    ```sh
    sudo umount ./efs-mount
    ```


