# Deploying Apache Airflow on AWS (with ECS Fargate)

This repository is inspired by the work of Axel Furlan and his [setup guide](https://towardsdatascience.com/how-to-deploy-apache-airflow-with-celery-on-aws-ce2518dbf631). 

## Step 1) Prepare Docker Image

- Clone [Puckel's Airflow](https://github.com/puckel/docker-airflow)
 - Modify entrypoint.sh<nolink>, airflow.cfg, Dockerfile, CeleryExecutor.yml or replace with files in this repo
 - Build new docker image: **docker build -t airflow . --no-cache**

## Step 2) Upload Docker Image (ECR)

- Navigate to [Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)
- Create repository
- Enter repository
- View push commands
- Follow instructions to authenticate your Docker client, tag image and push to ECR
- Verify ECR image

## Step 3) Create Cluster (ECS)

- Navigate to [Elastic Container Service (ECS)](https://aws.amazon.com/ecs/)
- Create Cluster (named airflow-celery)
- Select Networking Only (this guide will utilize Fargate tasks)
- Add Cluster name
- Create VPC with Subnet 1 only
- Create Cluster

## Step 4) Create PostGreSQL Database (RDS v9.6)

- Navigate to [Relational Database Service (RDS)](https://aws.amazon.com/rds/)
- Create database
- Standard Create
- Select PostgreSQL
- Select Version 9.6.16-R1
- Select Free tier
- Modify DB instance identifier
- Change username to airflow
- Make a unique password (to be used in Task Definition below)
- Within Connectivity, select the VPC of your ECR cluster
- Within Additional configuration, create Initial database name
- Create database
- Once created, navigate to database and make note of Endpoint URL (to be used in Task Definition below)



## Step 5) Create Task Definitions (ECS)

- Within [ECS](https://aws.amazon.com/ecs/) select Task Definitions
- Create 5 tasks (Flower, Redis, Scheduler, Webserver, Worker)

#### Flower
|Settings|     |
|----------------|-------------|
|Task Definition Name         |airflow-flower  |     
|Network Mode         |awsvpc  |  
|Compatabilities|EC2, FARGATE| 
|Requires compatabilities|FARGATE| 
|Task execution role|ecsTaskExecutionRole| 
|Task Size|512 MiB| 
|Task CPU|256| 

#### Flower Container Definitions
|Settings|     ||
|----------------|-------------|-------------|
|Image|ECR/airflow:latest| 
|Command|flower| 
|Host Port/Container Port|5555|TCP| 

#### Redis

|Settings|     |
|----------------|-------------|
|Task Definition Name         |airflow-redis  |     
|Network Mode         |awsvpc  |  
|Compatabilities|EC2, FARGATE| 
|Requires compatabilities|FARGATE| 
|Task execution role|ecsTaskExecutionRole| 
|Task Size|2048 MiB| 
|Task CPU|1024| 

#### Redis Container Definitions

|Settings|     ||
|----------------|-------------|-------------|
|Image|docker.io/redis:5.0.5| 
|Host Port/Container Port|6379|TCP| 

#### Redis Environment Variables

|Settings|     |
|----------------|-------------|
|POSTGRES_DB    |airflow-redis  |  
|POSTGRES_HOST    |(RDS Endpoint) |
|POSTGRES_PASSWORD    |(RDS Admin Password)| 
|POSTGRES_PORT    |5432  | 
|POSTGRES_USER    |airflow  | 

#### Scheduler

|Settings|     |
|----------------|-------------|
|Task Definition Name         |airflow-scheduler  |     
|Network Mode         |awsvpc  |  
|Compatabilities|EC2, FARGATE| 
|Requires compatabilities|FARGATE| 
|Task execution role|ecsTaskExecutionRole| 
|Task Size|2048 MiB| 
|Task CPU|512| 

#### Scheduler Container Definitions

|Settings|     |
|----------------|-------------|
|Image|ECR/airflow:latest| 
|Command|scheduler| 

#### Scheduler Environment Variables

|Settings|     |
|----------------|-------------|
|POSTGRES_DB    |airflow-scheduler  |  
|POSTGRES_HOST    |(RDS Endpoint) |
|POSTGRES_PASSWORD    |(RDS Admin Password)| 
|POSTGRES_PORT    |5432  | 
|POSTGRES_USER    |airflow  | 
|REDIS_HOST    |redis-service.local  | 

#### Webserver

|Settings|     |
|----------------|-------------|
|Task Definition Name         |airflow-scheduler  |     
|Network Mode         |awsvpc  |  
|Compatabilities|EC2, FARGATE| 
|Requires compatabilities|FARGATE| 
|Task execution role|ecsTaskExecutionRole| 
|Task Size|1024 MiB| 
|Task CPU|512| 

#### Webserver Container Definitions

|Settings|     ||
|----------------|-------------|-------------|
|Image|ECR/airflow:latest| 
|Command|webserver| 
|Host Port/Container Port|8080|TCP| 

#### Webserver Environment Variables

|Settings|     |
|----------------|-------------|
|POSTGRES_DB    |airflow-webserver  |  
|POSTGRES_HOST    |(RDS Endpoint) |
|POSTGRES_PASSWORD    |(RDS Admin Password)| 
|POSTGRES_PORT    |5432  | 
|POSTGRES_USER    |airflow  | 
|REDIS_HOST    |redis-service.local  | 

#### Worker

|Settings|     |
|----------------|-------------|
|Task Definition Name         |airflow-worker  |     
|Network Mode         |awsvpc  |  
|Compatabilities|EC2, FARGATE| 
|Requires compatabilities|FARGATE| 
|Task execution role|ecsTaskExecutionRole| 
|Task Size|3072 MiB| 
|Task CPU|1024| 

#### Worker Container Definitions

|Settings|     ||
|----------------|-------------|-------------|
|Image|ECR/airflow:latest| 
|Command|worker| 
|Host Port/Container Port|8793|TCP| 

#### Worker Environment Variables

|Settings|     |
|----------------|-------------|
|POSTGRES_DB    |airflow-worker  |  
|POSTGRES_HOST    |(RDS Endpoint) |
|POSTGRES_PASSWORD    |(RDS Admin Password)| 
|POSTGRES_PORT    |5432  | 
|POSTGRES_USER    |airflow  | 
|REDIS_HOST    |redis-service-master.local  | 

## Step 6) Create Services (ECS)

- Within [ECS](https://aws.amazon.com/ecs/) select Cluster -> Services -> Create
- Create 5 services (Flower, Redis, Scheduler, Webserver, Worker)

#### Flower Service
|Settings|     |
|----------------|-------------|
|Launch type|FARGATE |  
|Task Definition    |airflow-flower  |  
|Cluster    |airflow-celery|
|Service name|flower-service|
|Number of tasks|1| 
|Cluster VPC|(select VPC created with Cluster)| 
|Subnets|Subnet 1 (10.0.0.0/24)| 
|Security Groups|Master Security Group| 
|Auto-assign public IP|Enabled| 
#### Redis Service
|Settings|     |
|----------------|-------------|
|Launch type|FARGATE |  
|Task Definition    |airflow-redis  |  
|Cluster    |airflow-celery|
|Service name|redis-service|
|Number of tasks|1| 
|Cluster VPC|(select VPC created with Cluster)| 
|Subnets|Subnet 1 (10.0.0.0/24)| 
|Security Groups|Master Security Group| 
|Auto-assign public IP|Enabled| 
#### Scheduler Service
|Settings|     |
|----------------|-------------|
|Launch type|FARGATE |  
|Task Definition    |airflow-scheduler  |  
|Cluster    |airflow-celery|
|Service name|scheduler-service|
|Number of tasks|1| 
|Cluster VPC|(select VPC created with Cluster)| 
|Subnets|Subnet 1 (10.0.0.0/24)| 
|Security Groups|Master Security Group| 
|Auto-assign public IP|Enabled| 
#### Worker Service
|Settings|     |
|----------------|-------------|
|Launch type|FARGATE |  
|Task Definition    |airflow-worker  |  
|Cluster    |airflow-celery|
|Service name|worker-service|
|Number of tasks|1| 
|Cluster VPC|(select VPC created with Cluster)| 
|Subnets|Subnet 1 (10.0.0.0/24)| 
|Security Groups|Master Security Group| 
|Auto-assign public IP|Enabled| 
#### Webserver Service
|Settings|     |
|----------------|-------------|
|Launch type|FARGATE |  
|Task Definition    |airflow-webserver  |  
|Cluster    |airflow-celery|
|Service name|webserver-service|
|Number of tasks|1| 
|Cluster VPC|(select VPC created with Cluster)| 
|Subnets|Subnet 1 (10.0.0.0/24)| 
|Security Groups|Webserver Security Group| 
|Auto-assign public IP|Enabled| 
## Security Group Settings

Note: In order to view ECS logs within the AWS console, you will need to also enable HTTPS, Port 443, to 0.0.0.0/0


### Webserver Inbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  |(Master Security Group)                  |
|Custom TCP Rule|TCP|5432  | (PostgreSQL Security Group)     |
|Custom TCP Rule |TCP|8080     | (Your Static IP for Web UI Access)|

### Webserver Outbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  | (Master Security Group)|
|Custom TCP Rule|TCP|5432  | (PostgreSQL Security Group)     |
|Custom TCP Rule |TCP|8080     | (Your Static IP for Web UI Access)|

### Master Inbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  |(Master Security Group)|
|Custom TCP Rule|TCP|5432  | (PostgreSQL Security Group)     |

### Master Outbound Rules

|                |Protocol     |Port Range  |Source|
|----------------|-------------|------------|------|
|All TCP         |TCP|0-65535  | (Master Security Group)                   |
|Custom TCP Rule|TCP|5432  | (PostgreSQL Security Group)     |
