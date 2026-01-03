pipeline {
    agent any

    stages {

        stage('code-clone') {
            steps {
                git 'https://github.com/sonugupta4166/Docker-web-app.git'
            }
        }

        stage('clean-unused-files') {
            steps {
                sh '''
                rm -rf Docker-web ansible helm compose kubernetes README.md
                '''
            }
        }

        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('copy-war-to-docker-context') {
            steps {
                sh '''
                rm -rf Docker-app/target
                mkdir -p Docker-app/target
                cp target/*.war Docker-app/target/
                '''
            }
        }

        stage('image-build') {
            steps {
                sh 'docker build -t appimage ./Docker-app'
            }
        }

        stage('db-build') {
            steps {
                sh 'docker build -t dbimage ./Docker-db'
            }
        }

        stage('check-images') {
            steps {
                sh 'docker images'
            }
        }

        stage('dbimage-run') {
            steps {
                sh '''
                docker rm -f devopsdb || true
                docker run -d --name devopsdb -p 3306:3306 dbimage
                '''
            }
        }

        stage('appimage-run-with-link-db') {
            steps {
                sh '''
                docker rm -f devopsapp || true
                docker run -d \
                --name devopsapp \
                -p 8081:8080 \
                --link devopsdb:mysqlconnect \
                appimage
                '''
            }
        }

        stage('verify') {
            steps {
                sh 'docker ps'
            }
        }
    }
}
