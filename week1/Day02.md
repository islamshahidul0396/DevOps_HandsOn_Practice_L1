# Day 02 - Services & Tags

## Ansible - Install & Enable Nginx

- name: Install and start Nginx on Pathnex
  hosts: all
  become: yes

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present
    
    - name: Enable nginx
      service:
        name: nginx
        state: started
        enabled: yes

## Terraform - EC2 with Tags (r5.2xlarge)

provider "aws" {
    region = "us-east-1"
}

resource "aws_instance" "PathnexEC2" {
    ami = "ami-0abcd1234abcd1234"
    instance_type = "r5.2xlarge"

tags = {
    Name        = "Pathnex-Server"
    Environment = "Training"
    Owner       = "PathnexStudent"
}
}

## Kubernets - Deployment with 2 Replicas

apiVersion: apps/v1
kind: Deployment
metadata:
    name: pathnex-development
spec:
  replicas: 2
  selector:
        matchLabels:
          app: pathnex-app
  template:
    metadata:
        labels:
            app: pathnex-app
    spec:
        containers:
            - name: app
              image: nginx
              ports:
                - containerPort: 80

## Shell Script - Disk Usage

#! /bin/bash
df -h

# Maven Build
## Jenkins Pipeline - Maven Build
You will learn how to **compile, test, and package a Java project** using Maven.

pipeline{
    agent any
    tools {
        maven 'Maven-3.8.1'
        jdk 'JDK-17'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Compile') {
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
}

## Gitlab CI - Maven Build

stages:
  - build
  - test
  - package

maven-build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean compile

maven-test:
  stage: test
  image: maven:3.8.1-jdk-17
  script:
    - mvn test

maven-package:
  stage: package
  image: maven:3.8.1-jdk-17
  script:
    - mvn package
  artifacts:
    paths:
      - target/*.jar
      - target/*.war
    expire_in: 1 week