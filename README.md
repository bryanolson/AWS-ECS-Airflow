# Deploying Apache Airflow on AWS

This tutorial is built off of the work of Axel Furlan and his [amazing guide]([https://towardsdatascience.com/how-to-deploy-apache-airflow-with-celery-on-aws-ce2518dbf631](https://towardsdatascience.com/how-to-deploy-apache-airflow-with-celery-on-aws-ce2518dbf631)), but will expand upon on setup details that were pain points for my own project. 

## Step 1) Prepare Docker Image

- Fork Repo
 - Modify entrypoint.sh<nolink>, airflow.cfg, Dockerfile, CeleryExecutor.yml

## Step 2) Upload Docker Image (ECR)

- [Navigate to Elastic Container Registry (ECR)]([https://aws.amazon.com/ecr/](https://aws.amazon.com/ecr/))
- Create repository
- Enter repository
- View push commands
- Follow instructions to authenticate your Docker client, tag image and push to ECR
- Verify ECR image

## Step 3) Create Cluster (ECS)

- Navigate to [Elastic Container Service (ECS)]([https://aws.amazon.com/ecs/](https://aws.amazon.com/ecs/))
- Create Cluster
- Select Networking Only (this guide will utilize Fargate tasks)
- Add Cluster name
- Create VPC with Subnet 1 only
- Create Cluster

## Step 4) Create PostGreSQL Database (RDS v9.6)

- [Navigate to Relational Database Service (RDS)]([https://aws.amazon.com/rds/](https://aws.amazon.com/rds/))
- Create database
- Standard Create
- Select PostgreSQL
- Select Version 9.6.16-R1
- Select Free tier
- Modify DB instance identifier
- Change username to airflow
- Make a unique password
- Within Connectivity, select the VPC of your ECR cluster
- Within Additional configuration, create Initial database name
- Create database



## Step 5) Create Task Definitions (ECS)

- Within [ECS]([https://aws.amazon.com/ecs/](https://aws.amazon.com/ecs/)) select Task Definitions
- TODO

## Step 6) Create Services (ECS)

- TODO

## Security Group Settings

### Inbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  |(Master Security Group)                  |
|All TCP         |TCP|5432  | (PostgreSQL Security Group)     |
|Custom TCP Rule |TCP|8080     | (Static IP for Web UI Access)|

### Outbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  | (Master Security Group)                   |
|All TCP         |TCP|5432  | (PostgreSQL Security Group)     |
|Custom TCP Rule |TCP|8080     | (Static IP for Web UI Access)|
