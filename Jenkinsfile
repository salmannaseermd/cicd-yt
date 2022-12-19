pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
           
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube --stacktrace'
    
                    }

                    timeout(5) {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }

        }

        stage("docker build & docker push to nexus repository/registry"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                             sh '''
                                docker build -t 3.208.10.171:8083/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 3.208.10.171:8083 
                                docker push  3.208.10.171:8083/springapp:${VERSION}
                                docker rmi 3.208.10.171:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "msniron@gmail.com";  
		}
	}

}