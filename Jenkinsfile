def version
pipeline {
    agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Build') { 
            steps {
                version=$(git describe)
                sh 'mvn -B -DskipTests -Drevision=$version clean package'
            }
            
        }
        stage('Test') {
            steps {
                sh 'mvn test -Drevision=$version'
            }	
        }
	stage('Docker build') {
            agent any
            steps {
                sh 'docker build -t trialdaybitadmin/test-image:$version --build-arg JAR_FILE=target/*.jar'
	    }
        }
        stage('Docker Push') {
            agent any
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
               sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
               sh 'docker push trialdaybitadmin/test-image:$version'
        }
      }
    }

}
