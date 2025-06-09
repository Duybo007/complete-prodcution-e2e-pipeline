pipeline{
    agent{
        label "jenkins-agent"
    }

    tools {
        jdk "Java17"
        maven "Maven3"
    }

    stages{
        stage("Cleanup workspace"){
            steps{
                cleanWs()
            }
        }

        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Duybo007/complete-prodcution-e2e-pipeline.git'
            }
        }
    }
}