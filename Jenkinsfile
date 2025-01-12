pipeline{
    agent{
        label "jenkins-agent"
    }
    tools {
        jdk 'Java20'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-production-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "nadir24950"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}" + "-" + "${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    } 
    
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Nadir24950/complete-production-e2e-pipeline'
            }

        }
        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }
        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
        stage("SonarQube Analysis"){
            steps{
                withSonarQubeEnv(installationName:"sonarqube-scanner",credentialsId:"jenkins-sonarqube-token"){
                    sh "mvn sonar:sonar "
                }
            }
        }
        stage("Quality Gate"){
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'   
                }  
            }
        }
        stage("Build and push Docker Image"){
            steps{
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
        stage("Trigger CD Pipeline"){
            steps{
                script{
                    sh "curl -v -k --user Nadir24950:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.cloud-devops-project.dev/job/gitops-jenkins-java/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}