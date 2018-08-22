def ARTIFACT_VERSION
pipeline {
    agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Prepare') {
	  agent any
          steps {
             script {
                 ARTIFACT_VERSION = sh(returnStdout: true, script: 'git describe').trim()
             }
          }

        }
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests -Drevision=$ARTIFACT_VERSION clean package'
            }
            
        }
        stage('Test') {
            steps {
                sh 'mvn test -Drevision=$ARTIFACT_VERSION'
            }	
        }
	stage('Docker build') {
            agent any
            steps {
                sh 'docker build -t trialdaybitadmin/test-image:$ARTIFACT_VERSION --build-arg JAR_FILE=target/*.jar'
	    }
        }
        stage('Docker Push') {
            agent any
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
               sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
               sh 'docker push trialdaybitadmin/test-image:$ARTIFACT_VERSION'
               }
            }
        }
    }
}
