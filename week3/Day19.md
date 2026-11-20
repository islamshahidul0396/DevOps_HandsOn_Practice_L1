# Day 19 - Git Deploy, RDS Skeleton, Rolling Strategy, Ping Check

## Ansible - Deploy App Using Git

---
- name: Deploy app from Git for Pathnex
  hosts: all
  become: yes

  tasks:
    - name: Install Git
      yum:
        name: git
        state: present

    - name: Clone repository
      git:
        repo: "https://github.com/pathnex/app.git"
        dest: "/opt/pathnex-app"
        version: main
        force: yes


## Terraform - RDS Skeleton (MySQL)


resource "aws_db_instance" "PathnxRDS" {
  allocated_storage      = 20
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = "db.t3.micro"

  db_name                = "pathnexdb"
  username               = "admin"
  password               = var.db_password

  publicly_accessible    = false
  storage_encrypted      = true
  skip_final_snapshot    = true
  deletion_protection    = false

  backup_retention_period = 7

  tags = {
    Name = "Pathnex-RDS"
  }
}

variable "db_password" {
  description = "RDS master password"
  type        = string
  sensitive   = true
}

export TF_VAR_db_password="YourStrongPassword"

## Kubernetes - Deployment Rolling Update (Detailed)


apiVersion: apps/v1
kind: Deployment

metadata:
  name: pathnex-rollout

spec:
  replicas: 3

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1

  selector:
    matchLabels:
      app: rollout

  template:
    metadata:
      labels:
        app: rollout

    spec:
      containers:
        - name: app
          image: nginx:latest
          ports:
            - containerPort: 80


## Shell Script - Check Internet Connectivity


#!/bin/bash

ping -c 2 google.com &> /dev/null

if [ $? -eq 0 ]; then
    echo "Internet is available"
else
    echo "No internet connection"
fi


# Approvals

## Jenkins Pipeline - Manual Approval
You will learn how to **pause pipeline until manual approval**.


pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {
        stage('Deploy') {
            steps {
                input message: "Approve deployment for ${INSTITUTE_NAME}?",
                      ok: 'Approve'

                sh 'echo "Deployment approved and executed!"'
            }
        }
    }
}


## GitLab CI - Manual Approval Job
You will learn how to **use manual jobs in GitLab CI for approvals**.


stages:
  - build
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"

build:
  stage: build
  image: alpine:latest
  script:
    - echo "Building $INSTITUTE_NAME application"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME application"
    # Add Kubernetes deployment commands here
  when: manual
  allow_failure: false
