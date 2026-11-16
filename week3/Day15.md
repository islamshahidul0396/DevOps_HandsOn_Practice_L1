# Day 15 - Java, S3 Bucket, CronJob, File Reader

## Ansible - Install Java & Set JAVA_HOME

---

---
- name: Install Java on Pathnex servers
  hosts: all
  become: yes

  tasks:
    - name: Install Java 11
      package:
        name: "{{ 'java-11-openjdk' if ansible_os_family == 'RedHat' else 'openjdk-11-jdk' }}"
        state: present

    - name: Configure JAVA_HOME
      lineinfile:
        path: /etc/profile
        line: 'export JAVA_HOME={{ "/usr/lib/jvm/java-11-openjdk" if ansible_os_family == "RedHat" else "/usr/lib/jvm/java-11-openjdk-amd64" }}'
        state: present

    - name: Configure PATH
      lineinfile:
        path: /etc/profile
        line: 'export PATH=$JAVA_HOME/bin:$PATH'
        state: present

    - name: Verify Java installation
      command: java -version
      register: java_version
      changed_when: false

    - name: Display Java version
      debug:
        var: java_version.stderr


## Terraform - Create S3 Bucket


terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "pathnex_bucket" {
  bucket = "pathnex-devops-bucket"

  tags = {
    Name        = "PathnexBucket"
    Environment = "Dev"
    ManagedBy   = "Terraform"
  }
}


## Kubernetes - CronJOb

apiVersion: batch/v1
kind: CronJob
metadata:
    name: pathnex-cron
spec:
    schedule: "*/5 * * * *"
    jobTemplate:
        spec:
            template:
                spec:
                    containers:
                        - name: job
                          image: busybox
                          command: ["sh", "-c", "echo Pathnex CronJob running"]
                    restartPolicy: OnFailure

## Shell Script - Read File Line-by-Line


#!/bin/bash

FILE="/etc/passwd"

while IFS= read -r line; do
    echo "Line: $line"
done < "$FILE"



# Advanced Notifications

## Jenkins Pipeline - Slack Notification
You will learn how to **send Slack notifications after build sucess or failure**.


pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
        SLACK_CHANNEL = "#devops"
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

    post {
        success {
            slackSend(
                channel: "${env.SLACK_CHANNEL}",
                message: "SUCCESS: ${env.INSTITUTE_NAME} Build #${env.BUILD_NUMBER}"
            )
        }

        failure {
            slackSend(
                channel: "${env.SLACK_CHANNEL}",
                message: "FAILURE: ${env.INSTITUTE_NAME} Build #${env.BUILD_NUMBER}"
            )
        }
    }
}


## GitLab CI - Slack Notification
You will learn how to **notify Slack in GitLab CI**.


stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"

build:
  stage: build
  image: maven:3.8.1-jdk-17

  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean package

  after_script:
    - |
      if [ "$CI_JOB_STATUS" = "success" ]; then
        MESSAGE="Build SUCCESS for ${INSTITUTE_NAME}, Pipeline #${CI_PIPELINE_ID}"
      else
        MESSAGE="Build FAILED for ${INSTITUTE_NAME}, Pipeline #${CI_PIPELINE_ID}"
      fi

      curl -sS -X POST \
        -H "Content-Type: application/json" \
        --data "{\"text\":\"${MESSAGE}\"}" \
        "$SLACK_WEBHOOK"
