# Day 03 - Users, SGs, Env Vars
## Ansible - Create User

---
- name: Create Pathnex user
  hosts: all
  become: yes

  tasks:
    - name: Create user pathnex-admin
      ansible.builtin.user:
        name: pathnex-admin
        shell: /bin/bash
        state: present

## Terrafrom - Security Group (r6i.4xlarge EC2)


provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "PathnexSG" {
  name        = "PathnexSG"
  description = "Pathnex SG for SSH"

  ingress {
    description = "Allow SSH from trusted IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "PathnexSG"
  }
}

resource "aws_instance" "PathnexEC2" {
  ami           = "YOUR_VALID_AMI_ID"
  instance_type = "r6i.4xlarge"

  vpc_security_group_ids = [
    aws_security_group.PathnexSG.id
  ]

  tags = {
    Name = "Pathnex-EC2"
  }
}



## Kubernetes - Add Environment Variables

apiVersion: v1
kind: Pod
metadata:
  name: pathnex-env-pod
spec:
  containers:
    - name: app
      image: nginx:latest
      env:
        - name: APP_ENV
          value: "production"


## SHell Script - Check File Exists

#!/bin/bash

FILE="/tmp/pathnex.txt"

if [ -f "$FILE" ]; then
    echo "File Exists"
else
    echo "File does not exist"
fi



# NodeJS Build
## Jenkins Pipeline - NodeJS Build
You will learn how to **install dependencies, run tests, and build a NodeJS project**.


pipeline {
    agent any

    tools {
        nodejs 'NodeJS-24'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-node-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}




## GitLab CI - NodeJS Build

stages:
  - install
  - test
  - build

node-install:
  stage: install
  image: node:24
  script:
    - npm install
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

node-test:
  stage: test
  image: node:24
  script:
    - npm test

node-build:
  stage: build
  image: node:24
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
