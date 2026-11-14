# Day 12 - File Content, RDS Skeleton, Rolling Update, Backup

## Ansible - Ensure File With Content

---
- name: Create config file for Pathnex
  hosts: all
  become: true

  tasks:
    - name: Write content to configuration file
      ansible.builtin.copy:
        dest: /etc/pathnex.conf
        content: |
          Welcome to Pathnex DevOps Training
          Today is Day 12
        owner: root
        group: root
        mode: '0644'



## Terraform - RDS Skeleton


resource "aws_db_instance" "PathnexRDS" {
  allocated_storage      = 20
  engine                 = "mysql"
  instance_class         = "db.t3.micro"

  db_name                = "pathnexdb"
  username               = var.db_username
  password               = var.db_password

  storage_encrypted      = true
  publicly_accessible    = false
  skip_final_snapshot    = true
  deletion_protection    = false

  tags = {
    Name        = "Pathnex-RDS"
    Environment = "Development"
  }
}



## Kubernetes - Rolling Update Strategy

apiVersion: apps/v1
kind: Deployment
metadata:
    name: pathnex-rolling
spec:
    replicas: 3
    strategy:
        type: RollingUpdate
        rollingUpdate:
            maxSurge: 1
            maxunavailable: 1
    selector:
        matchLabels:
            app: roll
    template:
        metadata:
            labels:
                app: roll
        spec:
            containers:
                - name: web
                  image: nginx

## Shell Script - Backup Directory

#!/bin/bash

SOURCE="/var/log"
BACKUP_DIR="/backup"
DEST="$BACKUP_DIR/pathnex-$(date +%F).tar.gz"

# Create backup directory if it does not exist
mkdir -p "$BACKUP_DIR"

# Check source directory
if [ ! -d "$SOURCE" ]; then
    echo "ERROR: Source directory does not exist: $SOURCE"
    exit 1
fi

# Create compressed backup
if tar -czf "$DEST" -C /var log; then
    echo "SUCCESS: Backup created at: $DEST"
    ls -lh "$DEST"
else
    echo "ERROR: Backup failed!"
    exit 1
fi

# Multi-branch Pipelines

## Jenkins Pipeline - Multi-branch
You will learn how to **handle multiple branches in Jenkins**.

pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }

    post {
        success {
            echo "Build successful for ${env.INSTITUTE_NAME}"
        }

        failure {
            echo "Build failed for ${env.INSTITUTE_NAME}"
        }

        always {
            echo "Pipeline execution completed"
        }
    }
}

## GitLab CI - Branch Specific Jobs
You will learn how to **run jobs for specific branches in GitLab CI**.

stages:
  - build

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"

cache:
  key: maven-cache
  paths:
    - .m2/repository/

build-main:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean compile
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

build-dev:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean compile
  rules:
    - if: '$CI_COMMIT_BRANCH == "dev"'