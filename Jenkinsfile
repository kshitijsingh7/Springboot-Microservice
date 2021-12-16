pipeline {
    parameters { choice(name: 'microservice', choices: ['user-service', 'department-service', 'service-registry' , 'cloud-gateway' , 'hystrix-dashboard' , 'cloud-config-server'], description: '') }
    
    environment {
        registryCredential = 'dockerhub'
    }
    
    agent any

    stages {
        
        stage('Git Checkout') {
            steps {
                git branch: 'master', changelog: false, poll: false, url: 'https://github.com/kshitijsingh7/Springboot-Microservice.git'
            }
        }
        
        stage('Maven Package') {
            steps {
                 sh '''cd $microservice
                        mvn package'''
            }
        } 
        
        stage('SonarQube analysis') { 
             steps {
                withSonarQubeEnv('Sonar') { 
                    sh '''cd $microservice
                        mvn clean verify sonar:sonar -Dsonar.projectKey=$microservice'''
                }
             }
        }
        
        stage("SonarQube Quality Gate") {
            steps {
                   waitForQualityGate abortPipeline: true
                }
        }
        
        
        stage('Approval Stage') {
            steps {
                input 'Approve Workflow?'
            }
        }        
        
        
        stage('Build Image') {
            steps {
                    sh '''
                    cd $microservice
                    docker build -t d1247/$microservice:latest .
                    '''
                
            }
        }

        stage('Push image') {
           steps{
               script {
                docker.withRegistry( '', registryCredential ) {
                    sh 'docker push d1247/$microservice:latest'
                }
              }
           }
        }
        stage('Run Containers') {
            steps {
                sh 'docker-compose -p springboot-microservice up -d'
            }
        }
    }
}
