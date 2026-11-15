# Day 14 - MINI PROJECT

Complete these steps:

## Terraform - Create EC2 (any instance type)

- VPC
- Subnet
- Internet gateway
- Route Tbale
- EC2 (c5.xlarge / r5.2xlarge / r6i.4xlarge / c6i.8xlarge / c6a.12xlarge)

terraform-ec2/
│
├── provider.tf
├── variables.tf
├── main.tf
├── outputs.tf
└── terraform.tfvars

1. provider.tf ---->
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
2. variables.tf  ---->
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "ap-south-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string

  default = "c5.xlarge"

  validation {
    condition = contains([
      "c5.xlarge",
      "r5.2xlarge",
      "r6i.4xlarge",
      "c6i.8xlarge",
      "c6a.12xlarge"
    ], var.instance_type)

    error_message = "Choose a supported instance type from the allowed list."
  }
}

variable "ami_id" {
  description = "Amazon Linux 2023 AMI ID for the selected region"
  type        = string
}

variable "key_name" {
  description = "Existing EC2 key pair name"
  type        = string
}

variable "my_ip" {
  description = "Your public IP in CIDR format"
  type        = string
}

4. main.tf ---->
# 1. Create VPC
resource "aws_vpc" "PathnexVPC" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "Pathnex-VPC"
  }
}

# 2. Create Public Subnet
resource "aws_subnet" "PathnexSubnet" {
  vpc_id                  = aws_vpc.PathnexVPC.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true

  tags = {
    Name = "Pathnex-Public-Subnet"
  }
}

# 3. Create Internet Gateway
resource "aws_internet_gateway" "PathnexIGW" {
  vpc_id = aws_vpc.PathnexVPC.id

  tags = {
    Name = "Pathnex-IGW"
  }
}

# 4. Create Route Table
resource "aws_route_table" "PathnexRT" {
  vpc_id = aws_vpc.PathnexVPC.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.PathnexIGW.id
  }

  tags = {
    Name = "Pathnex-Route-Table"
  }
}

# 5. Associate Route Table with Subnet
resource "aws_route_table_association" "PathnexRTA" {
  subnet_id      = aws_subnet.PathnexSubnet.id
  route_table_id = aws_route_table.PathnexRT.id
}

# 6. Create Security Group
resource "aws_security_group" "PathnexSG" {
  name        = "Pathnex-SG"
  description = "Allow SSH access"
  vpc_id      = aws_vpc.PathnexVPC.id

  ingress {
    description = "SSH from my IP only"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Pathnex-SG"
  }
}

# 7. Create EC2 Instance
resource "aws_instance" "PathnexEC2" {
  ami                         = var.ami_id
  instance_type               = var.instance_type
  subnet_id                   = aws_subnet.PathnexSubnet.id
  vpc_security_group_ids      = [aws_security_group.PathnexSG.id]
  key_name                    = var.key_name
  associate_public_ip_address = true

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
  }

  tags = {
    Name = "Pathnex-EC2"
  }
}

5. Outpouts.tf  ---->
output "vpc_id" {
  value = aws_vpc.PathnexVPC.id
}

output "subnet_id" {
  value = aws_subnet.PathnexSubnet.id
}

output "ec2_instance_id" {
  value = aws_instance.PathnexEC2.id
}

output "ec2_public_ip" {
  value = aws_instance.PathnexEC2.public_ip
}

output "ec2_instance_type" {
  value = aws_instance.PathnexEC2.instance_type
}

6. terraform.tfvars ---->
aws_region    = "ap-south-1"

instance_type = "c5.xlarge"

ami_id        = "ami-xxxxxxxxxxxxxxxxx"

key_name      = "Pathnex-Key"

my_ip         = "YOUR_PUBLIC_IP/32"

## Ansible - Install Nginx & Deploy HTML File
---
- name: Install Nginx and Deploy HTML
  hosts: webservers
  become: yes

  vars:
    web_root: /usr/share/nginx/html

  tasks:

    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start and enable Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy HTML file
      ansible.builtin.copy:
        src: files/index.html
        dest: "{{ web_root }}/index.html"
        owner: root
        group: root
        mode: '0644'
      notify: Restart Nginx

  handlers:

    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

## Kubernetes - Create Deployment + Service

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
      protocol: TCP


## Shell - Print CPU, Memory, Disk

#!/bin/bash

echo "===================================="
echo "       SYSTEM RESOURCE MONITOR"
echo "===================================="

# 1. CPU Usage
echo ""
echo "CPU USAGE:"
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')
echo "CPU Utilization: ${CPU}%"

# 2. Memory Usage
echo ""
echo "MEMORY USAGE:"

free -m | awk '
/Mem:/ {
    total=$2
    used=$3
    available=$7
    usage=(used/total)*100

    printf "Total Memory: %d MB\n", total
    printf "Used Memory: %d MB\n", used
    printf "Available Memory: %d MB\n", available
    printf "Memory Utilization: %.2f%%\n", usage
}'

# 3. Disk Usage
echo ""
echo "DISK USAGE:"

df -h / | awk '
NR==2 {
    printf "Total Disk: %s\n", $2
    printf "Used Disk: %s\n", $3
    printf "Available Disk: %s\n", $4
    printf "Disk Utilization: %s\n", $5
}'

echo ""
echo "===================================="
echo "       MONITORING COMPLETED"
echo "===================================="


# Scheduled Pipelines

## Jenkins Pipeline - Scheduled Build
You will learn how to **schedule a Jenkins pipeline using cron**.

pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    triggers {
        cron('H 2 * * *') //every day at 2 AM
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

## GitLab CI - Scheduled Pipeline
You will learn how to **schedule GitLab CI pipelines via cron**.

stages:
    - build

build:
    stage: build
    image: maven:3.8.1-jdk-17
    script:
        - git clone https://github.com/Pathnex/sample-java-app.git
        - cd sample-java-app
        - mvn clean package