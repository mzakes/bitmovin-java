def artifactVersion
pipeline {
    agent none 
    stages {
        stage('Checkout') {
          agent any
          steps {
             checkout scm
             script {
                  artifactVersion = sh(returnStdout: true, script: 'git describe').trim()
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
                echo "${artifactVersion}"
                sh "mvn -B clean package -DskipTests -Drevision=${artifactVersion}"
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
                sh "mvn test -Drevision=${artifactVersion}"
            }	
        }
	stage('Docker build') {
            agent any
            steps {
                sh "docker build -t trialdaybitadmin/bitmovin-java:${artifactVersion} --build-arg JAR_FILE=target/bitmovin-java-${artifactVersion}.jar ."
	    }
        }
        stage('Docker Push') {
            agent any
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
               sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
               sh "docker push trialdaybitadmin/bitmovin-java:${artifactVersion}"
               }
            }
        }
    }
}
