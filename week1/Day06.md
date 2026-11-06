# Day 06 - DEBUG DAY

## Ansible (broken)

- name: Install nginx
  hosts: all
  become: yes
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present


## Terraform (broken)

resource "aws_instance" "BrokenEC2" {
    ami = "ami-oxyz"
    instance_type   = t2.micro
tags = {
    Name = "Broken"
}
}

# Kubernetes (broken)
apiVersion: v1
kind: Pod
metadata:
    name: pathnex
spec:
    containers:
        - name: app
          image: nginx

# Debugging Pipelines
## Jenkins Pipeline - Debugging
You will learn how to **fix broken stages and handle errors**.

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            steps {
                sh '''
                mvn clean compile || { 
                echo "Compile failed"
                exit 1
                }
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                mvn test || { echo "Tests failed"; exit 1;}
                '''
            }
        }
    }
}

## GitLab CI - Debugging
You will learn how to **handle job failures and exit codes**.

stages:
- build
- test

build:
    stage: build
    image: maven:3.8.1-jdk-17
    script:
        - git clone https://github.com/Pathnex/sample-java-app.git
        - cd sample-java-app
        - mvn clean compile || { echo "Compile failed"; exit 1; }

test:
    stage: test
    image: maven:3.8.1-jdk-17
    script:
        - cd sample-java-app
        - mvn test || { echo "Tests failed"; exit 1; }