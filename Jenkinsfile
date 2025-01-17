pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        REGISTRY = "kubeimran/vproappdock"
        REGISTRYCREDENTIAL = "dockerhub"
    }

    stages{

        stage('Fetch Code') {
            steps {
                git branch: 'paac', url: 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage("BUILD AND PUSH DOCKER IMAGE") {
            steps{
                script{
                    dockerImage = docker.build REGISTRY + ":V$BUILD_NUMBER"
                }
            }
        }

        stage("Upload Image to Docker Hub") {
            steps{
                script{docker.withRegistry("", REGISTRYCREDENTIAL){
                    dockerImage.push("V$BUILD_NUMBER")
                    dockerImage.push("latest")
                }
                }
            }
        }

        stage("Remove Unused docker image") {
            step{
                sh "docker rmi -f $dockerImage:V$BUILD_NUMBER"
            }
        }


    }


}
