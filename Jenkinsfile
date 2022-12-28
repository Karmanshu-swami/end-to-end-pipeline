pipeline {
	agent {	
		label 'java-pipeline-slave'
		}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/Karmanshu-swami/end-to-end-pipeline.git'
				}
			}

		stage("build") {
			steps {
				sh 'sudo mvn dependency:purge-local-repository'
				sh 'sudo mvn clean package'
				}
			}
		stage("Image") {
			steps {
				sh 'sudo docker build -t java-repo:$BUILD_TAG .'
				sh 'sudo docker tag java-repo:$BUILD_TAG karmanshu/pipeline-java:$BUILD_TAG'
				}
			}
				
	
//		stage("Docker Hub") {
//			steps {
//			withCredentials([string(credentialsId: 'docker_hub_passwd', variable: 'docker_hub_password_var')]) {
//				sh 'sudo docker login -u srronak -p ${docker_hub_password_var}'
//				sh 'sudo docker push srronak/pipeline-java:$BUILD_TAG'
//				}
//			}	
//
//		}
		stage("QAT Testing") {
			steps {
				sh 'sudo docker rm -f $(sudo docker ps -a -q)'
				sh 'sudo docker run -dit -p 8080:8080 karmanshu/pipeline-java:$BUILD_TAG'
				}
			}
		stage("testing website") {
			steps {
				retry(7) {
				sh 'curl --silent http://15.206.205.102:8080/java-web-app/ | grep -i "india" '
					}
				}
			}

		stage("Approval status") {
			steps {
				script {
					 Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                echo 'userInput: ' + userInput
					}
				}	
		}
		stage("Prod Env") {
			steps {
			 sshagent(['ubuntu-jenkins-slave']) {
	                    sh "ssh -o StrictHostKeyChecking=no ubuntu@13.126.20.221 sudo docker run  -d  -p  49153:8080  karmanshu/javatest-app:$BUILD_TAG"
				}
			}
		}
	}
}

