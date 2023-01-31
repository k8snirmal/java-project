pipeline{
    agent any
    stages{
        stage("Sonar code check"){
            agent{
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    }    
                }
            }
        }
    }
}