# Day 13 - DEBUG DAY

Fix the following broken files:

## Ansible (broken) Fixed

---
- name: Install Nginx
  hosts: all
  become: true

  tasks:
    - name: Install Nginx package
      ansible.builtin.yum:
        name: nginx
        state: present


## Terraform (broken) Fixed

resource "aws_instance" "BadEC2" {
  ami           = "ami-123"
  instance_type = "c5.large"

  tags = {
    Name = "Fixed"
  }
}

## Kubernetes (broken)

apiVersion: apps/v1
kind: Deployment

metadata:
  name: fixed

spec:
  replicas: 2

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
          image: nginx
          ports:
            - containerPort: 80

# Matrix Builds

## Jenkins Pipeline - Matrix Build
You will learn how to **run the same build on multiple environment**.

pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {
        stage('Build') {
            matrix {
                axes {
                    axis {
                        name 'JAVA_VERSION'
                        values '8', '11', '17'
                    }
                }

                stages {
                    stage('Build Application') {
                        steps {
                            sh "echo Building with Java ${JAVA_VERSION}"
                        }
                    }
                }
            }
        }
    }
}

## GitLab CI - Matix Jobs
You will learn how to **run jobs across multiple environments using matrix**.

stages:
  - build

build:
  stage: build

  parallel:
    matrix:
      - JAVA_VERSION: ["8", "11", "17"]

  image: "maven:3.8.1-jdk-${JAVA_VERSION}"

  script:
    - echo "Building with Java $JAVA_VERSION"
    - java -version
    - mvn clean compile