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
				git branch: 'main', changelog: false, poll: false, url: 'https://github.com/dhamatvivek20/Register-app-cicd.git'
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
					withSonarQubeEnv(credentialsId: 'sonar') {
    						sh 'mvn sonar:sonar'
					}
				}
			}
		}

		stage("Build Docker Image") {
			steps {
				script {
					sh 'docker build -t dhamatvivek/register-app:latest .'
				}
			}
		}

		stage("Push Docker Image") {
			steps {
				script {
					withDockerRegistry(credentialsId: 'dockerhubpwd') {
					sh 'docker push dhamatvivek/register-app:latest'
					}
				}
			}
		}

		stage ('Cleanup Artifacts') {
           		steps {
               			script {
                    			sh "docker rmi dhamatvivek/register-app:latest"
              			}
          		}
       		}
		
		stage('Deploy to Kubernetes') {
            		steps {
                    		withCredentials([
                        		file(credentialsId: 'cacrt', variable: 'CA_CERT_PATH'),
                        		file(credentialsId: 'clientkey', variable: 'CLIENT_KEY_PATH'),
                        		file(credentialsId: 'clientcrt', variable: 'CLIENT_CERT_PATH'),
                        		kubeconfigContent(credentialsId: 'K8cred', variable: 'KUBECONFIG_CONTENT')
                    ]) {
                        script {
                            def kubeconfig = env.KUBECONFIG_CONTENT
                            kubeconfig = kubeconfig.replaceAll('/home/vivek/.minikube/ca.crt', env.CA_CERT_PATH)
                            kubeconfig = kubeconfig.replaceAll('/home/vivek/.minikube/profiles/minikube/client.crt', env.CLIENT_CERT_PATH)
                            kubeconfig = kubeconfig.replaceAll('/home/vivek/.minikube/profiles/minikube/client.key', env.CLIENT_KEY_PATH)
                            writeFile file: 'kubeconfig', text: kubeconfig
                            env.KUBECONFIG = 'kubeconfig'
                            env.PATH = "/usr/local/bin/:" + env.PATH
                            sh 'echo $PATH'
                            sh 'kubectl get pods'
                            sh 'kubectl apply -f deployment-service.yaml'

                        		}
                    		}
                
            		}
        	}
		
	}

	post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                <div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
                """

                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'dhamatvivek2011@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html'
                )
            }
        }
    }
} Hey!! Vivek
