# Day 20 - DEBUG DAY

Fix these intentionally broken files:

## Ansible - Broken Fixed

---
- name: Install stuff
  hosts: all
  become: yes

  tasks:
    - name: Install Apache HTTPD
      yum:
        name: httpd
        state: present


## Terraform - Broken Fixed

resource "aws_instance" "Test" {
  ami           = "ami-123"
  instance_type = "c6i.8xlarge"
}

## Kubernetes - Broken

apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fixed
  template:
    metadata:
      labels:
        app: fixed
    spec:
      containers:
        - name: app
          image: nginx:latest
          ports:
            - containerPort: 80


# Secrets & Credentials

## Jenkins Pipeline - Using Credentials
You will learn how to **use credentials securely in Jenkins**.


pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }

        stage('Use Secret') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'PATHNEX_API_KEY',
                        variable: 'API_KEY'
                    )
                ]) {
                    sh '''
                        echo "Using secret for $INSTITUTE_NAME"
                        ./deploy.sh
                    '''
                }
            }
        }
    }
}


## GitLab CI - Using CI/CD Variables
You will learn how to **use GitLab CI variables securely**.


stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"

build:
  stage: build
  image: alpine:latest
  script:
    - echo "Starting build for $INSTITUTE_NAME"
    - echo "API key is configured"
    - |
      if [ -z "$API_KEY" ]; then
        echo "API_KEY is missing"
        exit 1
      fi
