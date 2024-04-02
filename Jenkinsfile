pipeline {
	agent any
	tools {
		jdk 'jdk11'
		maven 'maven3'
	}
	stages {
		stage("cleanup Workspace"){
			steps {
				cleanWs()
			}
		}
		
		stage("Checkout from SCM"){
			steps {
				git branch: 'main', changelog: false, poll: false, url: 'https://github.com/dhamatvivek20/register-app.git'
			}
		}

		stage("Build Application"){
			steps {
				sh 'mvn clean package'
			}
		}

		stage("Test Application") {
			steps {
				sh 'mvn test'
			}
		}
		stage("SonarQube Analysis") {
			steps {
				script {
					withSonarQubeEnv(credentialsId: '6e88d42c-041b-47b8-89ee-26c83003cc8a') {
    						sh 'mvn sonar:sonar'
					}
				}
			}
		}

		stage("Build Docker Image") {
			steps {
				script {
					sh 'docker build -t dhamatvivek/register-app .'
				}
			}
		}

		stage("Push Docker Image") {
			steps {
				script {
					withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'docker-cred')]) {
    						sh 'docker login -u dhamatvivek -p ${docker-cred}'
					}
   					
					sh 'docker push dhamatvivek/register-app'
				}
			}
		}
		
	}
}
