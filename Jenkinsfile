pipeline {
    // Specify the Jenkins agent (node) where the pipeline will run
    agent {
        label "jenkins-agent" // This refers to a Jenkins node labeled "jenkins-agent"
    }

    // Define the tools needed during the pipeline execution
    tools {
        jdk "Java17"         // Use Java 17
        maven "Maven3"       // Use Maven 3
    }

    // Define global environment variables accessible throughout the pipeline
    environment {
        APP_NAME = "complete-production-e2e-pipeline"        // Name of the application
        RELEASE = "1.0.0"                                     // Release version
        DOCKER_USER = "duybo95"                              // Docker Hub username
        DOCKER_PASS = "dockerhub"                            // Jenkins credentials ID for Docker Hub password
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"            // Full image name for Docker
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"             // Image tag including Jenkins build number
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN') // Jenkins API token used to trigger other jobs
    }
    

    stages {
        stage("Cleanup workspace") {
            steps {
                cleanWs() // Clean the Jenkins workspace before starting
            }
        }

        stage("Checkout from SCM") {
            steps {
                // Clone the code from the specified GitHub repository
                git branch: 'main', 
                    credentialsId: 'github', 
                    url: 'https://github.com/Duybo007/complete-prodcution-e2e-pipeline.git'
            }
        }

        stage("Build Application") {
            steps {
                // Compile the Java project using Maven
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                // Run unit tests using Maven
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    // Run SonarQube static code analysis using Maven
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    // Wait for the SonarQube quality gate result before proceeding
                    waitForQualityGate abortPipeline: false, 
                                       credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repo
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    // Push the built Docker image with versioned and latest tags
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    // Trigger the downstream Jenkins CD job with the Docker image tag
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST \
                        -H 'cache-control: no-cache' \
                        -H 'content-type: application/x-www-form-urlencoded' \
                        --data 'IMAGE_TAG=${IMAGE_TAG}' \
                        'http://172.31.1.73:8080/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}
