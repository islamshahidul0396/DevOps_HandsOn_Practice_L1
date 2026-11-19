# Day 18 - Facts, Variables, Affinity, Log Archiving

## Ansible - Use Ansible Facts

---
- name: Show system facts for Pathnex
  hosts: all
  become: yes
  gather_facts: yes

  tasks:
    - name: Print OS info
      debug:
        var: ansible_facts['os_family']

    - name: Print IP address
      debug:
        var: ansible_facts['default_ipv4']['address']

## Terraform - Variables, Outputs, tfvars

#variables.tf
variable "instance_type" {
    default = "c6a.12xlarge"
}

# main.tf
resource "aws_instance" "PathnexServer" {
    ami     = "ami-0abcd1234abcd1234"
    instance_type = var.instance_type
}

#output
output "pathnex_ip" {
    value = aws_instance.PathnexServer.public_ip
}

## Kubernetes - Node Affinity

apiVersion: v1
kind: Pod

metadata:
  name: pathnex-affinity

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd

  containers:
    - name: web
      image: nginx

## Shell Script - Archive Log Files

#!/bin/bash

BACKUP_DIR="/backup"
SOURCE_DIR="/var/log"
DATE=$(date +%F)
BACKUP_FILE="$BACKUP_DIR/pathnex-logs-$DATE.tar.gz"

# Create backup directory if missing
mkdir -p "$BACKUP_DIR"

# Create compressed archive
if tar -czf "$BACKUP_FILE" "$SOURCE_DIR"; then
    echo "Logs archived successfully: $BACKUP_FILE"
else
    echo "Log backup failed!"
    exit 1
fi

# Rollback Simulation

## Jenkins Pipeline - Rollback on Failure
pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
        DEPLOYMENT_NAME = "pathnex-app"
        NAMESPACE = "default"
        IMAGE = "nginx:1.27"
    }

    stages {

        stage('Deploy') {
            steps {
                script {
                    try {
                        echo "Deploying ${INSTITUTE_NAME}..."

                        sh """
                        kubectl set image deployment/${DEPLOYMENT_NAME} \
                        web=${IMAGE} -n ${NAMESPACE}

                        kubectl rollout status deployment/${DEPLOYMENT_NAME} \
                        -n ${NAMESPACE} --timeout=120s
                        """

                    } catch (Exception e) {

                        echo "Deployment failed!"
                        echo "Rolling back ${INSTITUTE_NAME}..."

                        sh """
                        kubectl rollout undo deployment/${DEPLOYMENT_NAME} \
                        -n ${NAMESPACE}

                        kubectl rollout status deployment/${DEPLOYMENT_NAME} \
                        -n ${NAMESPACE} --timeout=120s
                        """

                        error("Deployment failed; rollback was attempted.")
                    }
                }
            }
        }
    }
}

## GitLab CI - Rollback Simulation
You will learn how to **simulate rollback using after_script**.

stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"
  NAMESPACE: "default"
  DEPLOYMENT_NAME: "pathnex-app"

deploy:
  stage: deploy
  image: bitnami/kubectl:latest

  script:
    - echo "Deploying $INSTITUTE_NAME application..."
    - kubectl set image deployment/$DEPLOYMENT_NAME web=nginx:1.27 -n $NAMESPACE
    - kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=120s

  after_script:
    - |
      if [ "$CI_JOB_STATUS" = "failed" ]; then
        echo "Deployment failed. Rolling back $INSTITUTE_NAME..."
        kubectl rollout undo deployment/$DEPLOYMENT_NAME -n $NAMESPACE
        kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=120s
      fi
