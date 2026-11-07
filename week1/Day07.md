# Day 07 - SPEED TYPING

Rewrite ALL FOUR from scratch:

1. Ansible: Install nginx
2. Terraform: EC2 (choose any Pathnex instance)
3. Kubernetes: Deployment 2 replicas
4. Shell: Print date + uptime

Set a timer: **20 minutes**

# Parameterized Builds

## Jenkins Pipeline - Parameterized
You will learn how to **create parameterized builds in Jenkins**.

pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Deployment Environment')
    }
    environment {
        INSTITUE_NAME "Pathnex"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo "Deploying to $ENVIRONMENT by $INSTITUE_NAME"'
            }
        }
    }
}

## GitLab CI - Parameterized via Variables
You will learn how to **use variables to parameterized GitLab jobs**.


Stages:
    - deploy

variables:
    INSTITUE_NAME: "Pathnex"
    ENVIRONMENT: "dev"

deploy:
    stgae: deploy
    image: alpine:latest
    script:
        - echo "Deploying to $ENVIRONMENT by $INSTITUTE_NAME"

## Kubernetes Deployment - Parameterized via Variables
You will learn how to **use variables to parameterized Kubernetes Deployment**.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pathnex-deployment
  labels:
    app: pathnex
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pathnex
  template:
    metadata:
      labels:
        app: pathnex
    spec:
      containers:
        - name: pathnex
          image: pathnex:latest
          ports:
            - containerPort: 8080
          env:
            - name: APP_NAME
              value: "Pathnex"
            - name: INSTITUTE_NAME
              value: "Your Institute Name"

## Shell: Print date + uptime using Shell Script
You will elarn how to **print date and uptime using shell script**.

#!/bin/bash

echo "==== System Information ===="

echo "Current Date and Time:"
date

echo ""
echo "System Uptime:"
uptime

echo "=================================="

# Give execute permission
chmod +x script.sh

# Run the script
./script.sh