pipeline {
    environment {
      ARTIFACT_VERSION = ''
    }
    agent none 
    stages {
        stage('Checkout') {
          agent any
          steps {
             checkout scm
             script {
                  def artifactVersion= sh(returnStdout: true, script: 'git describe').trim()
                  env.ARTIFACT_VERSION = artifactVersion
                  echo ${env.ARTIFACT_VERSION}
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
                echo "{env.ARTIFACT_VERSION}"
                sh 'mvn -B -DskipTests -Drevision="${env.ARTIFACT_VERSION}" clean package'
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
                sh 'mvn test -Drevision="${env.ARTIFACT_VERSION}"'
            }	
        }
	stage('Docker build') {
            agent any
            steps {
                sh 'docker build -t trialdaybitadmin/test-image:"${env.ARTIFACT_VERSION}" --build-arg JAR_FILE=target/*.jar'
	    }
        }
        stage('Docker Push') {
            agent any
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
               sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
               sh 'docker push trialdaybitadmin/test-image:"${env.ARTIFACT_VERSION}"'
               }
            }
        }
    }
}
