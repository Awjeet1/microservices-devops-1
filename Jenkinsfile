pipeline {
    agent any
    
    stages{
        stage('SCA with OWASP Dependency Check') {
        steps {
            dependencyCheck additionalArguments: '''--format HTML
            ''', odcInstallation: 'DP-Check'
            }
    }

        stage('SonarQube Analysis') {
      steps {
        script {
          // requires SonarQube Scanner 2.8+
          scannerHome = tool 'SonarScanner'
        }
        withSonarQubeEnv('Sonarqube Server') {
          bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=newsread-microservices-application"
        }
      }
    }

        stage('Build Docker Images') {
            steps {
                script{
                    bat 'docker build -t awjeet777/newsread-customize customize-service/'
                    bat 'docker build -t awjeet777/newsread-news news-service/'
            }
        }
    }
        stage('Containerize And Test') {
            steps {
                script{
                    bat 'docker run -d  --name customize-service -e FLASK_APP=run.py awjeet777/newsread-customize && sleep 10 && docker logs customize-service && docker stop customize-service'
                    bat 'docker run -d  --name news-service -e FLASK_APP=run.py awjeet777/newsread-news && sleep 10 && docker logs news-service && docker stop news-service'
                }
            }
        }
        stage('Push Images To Dockerhub') {
            steps {
                    script{
                        withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubPass')]) {
                        bat 'docker login -u awjeet777 --password ${DockerHubPass}' }
                        bat 'docker push awjeet777/newsread-news && docker push awjeet777/newsread-customize'
               }
            }
                 
            }

        //stage('Trivy scan on Docker images'){
          //  steps{
            //     bat 'TMPDIR=/home/jenkins'
              //   bat 'trivy image awjeet777/newsread-news:latest'
                // bat 'trivy image awjeet777/newsread-customize:latest'
        //}
       
   // }
        }    

        post {
        always {
            // Always executed
                bat 'docker rm news-service'
                bat 'docker rm customize-service'
        }
        success {
            // on sucessful execution
            bat 'docker logout'   
        }
    }
}
