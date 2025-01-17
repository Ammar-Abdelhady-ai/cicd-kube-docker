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
                git branch: 'main', url: 'https://github.com/Ammar-Abdelhady-ai/cicd-kube-docker.git'
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

         stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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

        stage("Kubernetes Deploy"){
            steps{
                script{
                    sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${REGISTRY}:V${BUILD_NUMBER} --namespace prod"
                }
            }
        }


    }


}
