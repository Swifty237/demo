pipeline {
    agent any
    
    environment {
        registry = 'yannick237/demo'
        ip_prod = '13.38.216.87'
        port_prod = '8080'
        port_test = '8180'
        port_app = '8080'
        container_name = 'demo-boot'
    }

    stages {
        
         stage('Github checkout') {
            steps {
                echo 'Cloning demo application from github'
                git branch: 'main', 
                url: 'https://github.com/Swifty237/demo.git'
            }
        }
        
        stage('Compile code') {
            steps {
                echo 'Using maven to compile java application'
                sh 'mvn clean compile'
            }
        }
        
        stage('Tests Unit') {
            steps {
                echo 'Using maven to test java application'
                sh 'mvn test'
            }
        }
        
        stage('Build artifact jar') {
            steps {
                echo 'Using maven to build artifact of demo application'
                sh 'mvn package -DskipTests=true'
                archiveArtifacts 'target/*.jar'
            }
        }
        
        stage('Docker build and push') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub', url: '']) {
                    sh 'docker build -t $registry:$BUILD_NUMBER .'
                    sh 'docker push $registry:$BUILD_NUMBER'
                }
            }
        }
        
        stage('Removes unused docker image') {
            steps {
                sh 'docker rmi $registry:$BUILD_NUMBER -f'
            }
        }
        
        stage('Deploy to test env') {
            steps {
                sh 'docker stop $container_name || true'
                sh 'docker rm $container_name || true'
                sh 'docker run -d -p $port_test:$port_app --name $container_name $registry:$BUILD_NUMBER'
            }
        }
        
        stage('Going to Prod deployment') {
            steps {
                input 'Do you approve this prod deployment ?'
                echo 'Deploy to production...'
            }
        }
        
        stage('Deploy to AWS') {
            steps {
                sh 'docker -H $ip_prod stop $container_name || true'
                sh 'docker -H $ip_prod rm $container_name || true'
                sh 'docker -H $ip_prod run -d -p $port_prod:$port_app --name $container_name $registry:$BUILD_NUMBER'
            }
        }
    }
}
