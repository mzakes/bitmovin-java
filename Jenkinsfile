def ARTIFACT_VERSION
pipeline {
    agent none 
    stages {
        stage('Checkout') {
          steps {
             checkout scm
             script {
                 ARTIFACT_VERSION = sh(returnStdout: true, script: 'git describe --tags').trim()
             }
          }

        }
        stage('Build') {
            agent { 
              docker {
                image 'maven:3.5-alpine'
                args '-v /root/.m2:/root/.m2'            
              }
            } 
            steps {
                sh 'mvn -B -DskipTests -Drevision=$ARTIFACT_VERSION clean package'
            }
            
        }
        stage('Test') {
            agent { 
              docker {
                image 'maven:3.5-alpine'
                args '-v /root/.m2:/root/.m2'            
              }
            }
            steps {
                sh 'mvn test -Drevision=$ARTIFACT_VERSION'
            }	
        }
	stage('Docker build') {
            steps {
                sh 'docker build -t trialdaybitadmin/test-image:$ARTIFACT_VERSION --build-arg JAR_FILE=target/*.jar'
	    }
        }
        stage('Docker Push') {
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
               sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
               sh 'docker push trialdaybitadmin/test-image:$ARTIFACT_VERSION'
               }
            }
        }
    }
}
