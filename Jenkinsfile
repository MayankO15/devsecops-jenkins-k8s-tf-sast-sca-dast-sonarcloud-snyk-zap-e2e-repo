pipeline {
    agent any

    tools {
        maven 'Maven_3_8_4'
    }

    stages {

        stage('Compile and Run Sonar Analysis') {
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                -Dsonar.projectKey=MayankO15 \
                -Dsonar.organization=MayankO15 \
                -Dsonar.host.url=https://sonarcloud.io \
                -Dsonar.token=cb390e465e98126aa74adaa0c99bfece9cffdf14
                '''
            }
        }

        stage('Run SCA Analysis Using Snyk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerlogin', url: '']) {
                    script {
                        app = docker.build("mayank")
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    docker.withRegistry(
                        '079819354494.dkr.ecr.us-west-2.amazonaws.com/mayank',
                        'ecr:us-west-2:aws-credentials'
                    ) {
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh '''
                    echo "Current Context"
                    kubectl config current-context

                    echo "Nodes"
                    kubectl get nodes

                    echo "Deploying"
                    kubectl delete all --all -n devsecops || true
                    kubectl apply -f deployment.yaml -n devsecops

                    echo "Pods"
                    kubectl get pods -n devsecops

                    echo "Services"
                    kubectl get svc -n devsecops
                    '''
                }
            }
        }

        stage('Wait for Testing') {
            steps {
                sh '''
                pwd
                sleep 180
                echo "Application has been deployed on K8S"
                '''
            }
        }

        
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/jagpsi --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 
  }
}




// pipeline {
//   agent any
//   tools { 
//         maven 'Maven_3_8_4'  
//     }
//    stages{
//     stage('CompileandRunSonarAnalysis') {
//             steps {	
// 		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=jagpsi -Dsonar.organization=jagpsi -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=5e3f77bafff78138c2427798e42b4c1232da9502'
// 			}
//     }

// 	stage('RunSCAAnalysisUsingSnyk') {
//             steps {		
// 				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
// 					sh 'mvn snyk:test -fn'
// 				}
// 			}
//     }

// 	stage('Build') { 
//             steps { 
//                withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
//                  script{
//                  app =  docker.build("jagpsi")
//                  }
//                }
//             }
//     }

// 	stage('Push') {
//             steps {
//                 script{
//                     docker.withRegistry('https://139713842946.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
//                     app.push("latest")
//                     }
//                 }
//             }
//     	}
	   
// 	stage('Kubernetes Deployment of ASG Bugg Web Application') {
// 	   steps {
// 	      withKubeConfig([credentialsId: 'kubelogin']) {
// 		  sh('kubectl delete all --all -n devsecops')
// 		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
// 		}
// 	      }
//    	}
	   
// 	stage ('wait_for_testing'){
// 	   steps {
// 		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
// 	   	}
// 	   }
	   
// 	stage('RunDASTUsingZAP') {
//           steps {
// 		    withKubeConfig([credentialsId: 'kubelogin']) {
// 				sh('zap.sh -cmd -quickurl http://$(kubectl get services/jagpsi --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
// 				archiveArtifacts artifacts: 'zap_report.html'

// 		    }
// 	     }
//        } 
//   }
// }
