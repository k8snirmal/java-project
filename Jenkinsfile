pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar code check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
                steps{
                    script{
                        withSonarQubeEnv(credentialsId: 'sonar-pwd') {
                            sh "chmod +x gradlew"
                            sh "./gradlew sonarqube"
                        }
                        timeout(5) {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "the quality gate treshold: ${qg.status}"
                            }
                        }
                    }
                }
        }
        stage("docker build & push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                            docker build -t 34.100.190.138:8083/springapp:${VERSION} .
                            docker login -u admin -p $docker_password 34.100.190.138:8083
                            docker push 34.100.190.138:8083/springapp:${VERSION}
                            docker rmi 34.100.190.138:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
        stage("datree check"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=64351165-d9b2-4682-8b68-52dbe3993811']) {
                            sh "helm datree test myapp/ --debug"
                        }                     
                    }      
                }
            }
        }    
        stage("helm push-nexus"){
            steps{
                script{
                     withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        dir('kubernetes/') {
                            sh '''
                                helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf  myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_password http://34.100.190.138:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                           '''
                        }    
                    }                     
                }      
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'credentialsId', namespace: '', restrictKubeConfigAccess: false, serverUrl: 'https://10.160.0.16:6443'){
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.100.190.138:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                   }    
               }
            }
        }
    }        
}