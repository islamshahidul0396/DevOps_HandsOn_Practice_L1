# Day 16 - Ansible Roles, DynamoDB, Statefulset, Service Check

## Ansible - Create Role Structure

# Students must create:
# roles/
#    pathnex/
#      tasks/main.yml
---
- name: Install Apache
  apt:
    name: "{{ web_package }}"
    state: present
    update_cache: yes

- name: Start Apache
  service:
    name: "{{ web_service }}"
    state: started
    enabled: yes
#      handlers/main.yml
---
- name: Restart Apache
  service:
    name: apache2
    state: restarted
#      templates/index.html.j2
<html>
<head>
    <title>{{ institute_name }}</title>
</head>
<body>
    <h1>Welcome to {{ institute_name }}</h1>
    <h2>{{ training_message }}</h2>
    <p>Today is {{ ansible_date_time.date }}</p>
</body>
</html>
#      vars/main.yml
---
web_package: apache2
web_service: apache2

## Terraform - DynamoDB Table

resource "aws_dynamodb_table" "PathnexTable" {
  name         = "PathnexTrainingTable"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "ID"

  attribute {
    name = "ID"
    type = "S"
  }

  tags = {
    Name = "Pathnex-DDB"
  }
}

## Kubernetes - StatefulSet

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pathnex-db

spec:
  serviceName: pathnex
  replicas: 2

  selector:
    matchLabels:
      app: pathnex-db

  template:
    metadata:
      labels:
        app: pathnex-db

    spec:
      containers:
        - name: db
          image: mysql:8.0
          ports:
            - containerPort: 3306

          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "Pathnex123"

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: mysql-storage
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi

## Shell Script - Check Service Running

#!/bin/bash

SERVICE="sshd"

if systemctl is-active --quit $SERVICE; then
    echo "$SERVICE is running"
else
    echo "$SERVICE is NOT running"
fi

#!/bin/bash

SERVICES=("ssh" "nginx" "docker")

for SERVICE in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$SERVICE"; then
        echo "$SERVICE is running"
    else
        echo "WARNING: $SERVICE is NOT running"
    fi
done

# Artifacts Promotion 

## Jenkins Pipeline - Promote Artifacts
You will learn how to **promote build artifacts from dev to staging**.

pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Promote') {
            steps {
                input message: 'Promote artifacts to staging?',
                      ok: 'Approve'

                sh 'echo "Promoting artifacts for $INSTITUTE_NAME"'
            }
        }
    }
}

## GitLab CI - Artifacts Promotion
You will learn how to **promote artifacts manually using GitLab CI**.

stages:
  - build
  - promote

variables:
  INSTITUTE_NAME: "Pathnex"

build:
  stage: build
  image: maven:3.8.1-jdk-17

  script:
    - mvn clean package

  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week

promote:
  stage: promote
  image: alpine:latest

  script:
    - echo "Promoting artifacts for $INSTITUTE_NAME"

  when: manual
