# Day 17 - Loops, ALB, Namespace, Top Processes

## Ansible - Create Multiple Users in Loop


---
- name: Create multiple Pathnex users
  hosts: all
  become: yes

  vars:
    users:
      - dev1
      - dev2
      - dev3

  tasks:
    - name: Create users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop: "{{ users }}"


## Terraform - Application Load Balancer


resource "aws_lb" "PathnexALB" {
  name               = "pathnex-alb"
  internal           = false
  load_balancer_type = "application"

  security_groups = [
    aws_security_group.PathnexSG.id
  ]

  subnets = [
    aws_subnet.PathnexSubnet1.id,
    aws_subnet.PathnexSubnet2.id
  ]

  tags = {
    Name = "Pathnex-ALB"
  }
}


## Kubernetes - Namespace + Deployment


apiVersion: v1
kind: Namespace
metadata:
  name: pathnex-namespace

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pathnex-deploy
  namespace: pathnex-namespace
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
        - name: app
          image: nginx:latest
          ports:
            - containerPort: 80


## Shell Script - Top 5 Memory Processes
#!/bin/bash

ps aux --sort=-%mem | head -n 6

# Deployment Simulation

## Jenkins Pipeline - Simulated Deployment
You will learn how to **simulate deployment to different environments**.


pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'production'],
            description: 'Deployment Environment'
        )
    }

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {
        stage('Deploy') {
            steps {
                sh 'echo "Deploying $INSTITUTE_NAME application to $ENVIRONMENT"'
            }
        }
    }
}


## GitLab CI - Deployment Simulation
You will learn how to **simulate deployment in GitLab CI using environment varibale**.


stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"
  ENVIRONMENT: "staging"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME application to $ENVIRONMENT"
