🚀 Dockerized Java Web Application – CI/CD with Jenkins

This repository demonstrates a complete CI/CD pipeline for deploying a Java web application using Jenkins, Docker, MySQL, and Tomcat on a Linux server (AWS EC2).

The pipeline automates:

Source code checkout

Maven build

Docker image creation

Database initialization

Application deployment

📌 Tech Stack

Language: Java

Build Tool: Maven

CI/CD: Jenkins

Containerization: Docker

Web Server: Apache Tomcat 9 (Java 17)

Database: MySQL 5.7

OS: Linux (AWS EC2)

📁 Project Structure
Docker-web-app/
├── Docker-app/
│   └── Dockerfile              # Application Dockerfile
├── Docker-db/
│   ├── Dockerfile              # MySQL Dockerfile
│   └── db_backup.sql           # Database initialization script
├── Docker-web/
├── ansible/
├── compose/
├── helm/
├── kubernetes/
├── src/                         # Java source code
├── pom.xml                      # Maven configuration
├── README.md

⚙️ Prerequisites

Make sure the following are installed on the Jenkins server:

Java 17

Maven

Docker

Jenkins

Git

Required Ports (EC2 Security Group)
Port	Purpose
22	SSH
8081	Application
3306	MySQL
🐳 Docker Configuration
Application Dockerfile (Docker-app/Dockerfile)

Uses Tomcat 9 + Java 17

Deploys WAR file as ROOT.war

Exposes port 8080

Database Dockerfile (Docker-db/Dockerfile)
FROM mysql:5.7.25

ENV MYSQL_ROOT_PASSWORD=devopspassword
ENV MYSQL_DATABASE=accounts

ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql


Initializes database automatically on first run

🔄 CI/CD Pipeline Workflow

Clone source code from GitHub

Remove unused folders

Build Java application using Maven

Copy WAR file into Docker build context

Build Docker images (App & DB)

Run MySQL container

Run Application container

Verify running containers

🧩 Jenkins Pipeline

pipeline {
    agent any

    stages {

        stage('Code Clone') {
            steps {
                git 'https://github.com/sonugupta4166/Docker-web-app.git'
            }
        }

        stage('Clean Files') {
            steps {
                sh 'rm -rf Docker-web ansible helm compose kubernetes README.md'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Copy WAR to Docker Context') {
            steps {
                sh '''
                rm -rf Docker-app/target
                mkdir -p Docker-app/target
                cp target/*.war Docker-app/target/
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t appimage ./Docker-app
                docker build -t dbimage ./Docker-db
                '''
            }
        }

        stage('Run DB Container') {
            steps {
                sh '''
                docker rm -f devopsdb || true
                docker run -d --name devopsdb -p 3306:3306 dbimage
                '''
            }
        }

        stage('Run App Container') {
            steps {
                sh '''
                docker rm -f devopsapp || true
                docker run -d --name devopsapp -p 8081:8080 --link devopsdb:mysqlconnect appimage
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }
}



🧠 Key Learnings

Docker build context must include required artifacts

WAR file must be inside Docker build directory

Jenkins stops pipeline execution on failure

MySQL initialization scripts run only once

CI/CD reduces manual deployment effort
