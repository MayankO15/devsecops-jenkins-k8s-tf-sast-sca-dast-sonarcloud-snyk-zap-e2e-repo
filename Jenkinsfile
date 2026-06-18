pipeline {
agent any

```
tools {
    maven 'Maven_3_8_4'
}

environment {
    IMAGE_NAME = "mayank"
}

stages {

    stage('Create Namespace') {
        steps {
            withKubeConfig([credentialsId: 'kubelogin']) {
                sh '''
                    kubectl get namespace devsecops || kubectl create namespace devsecops
                '''
            }
        }
    }

    stage('Compile and Sonar Analysis') {
        steps {
            withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=mayanko15 \
                    -Dsonar.organization=mayanko15 \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=$SONAR_TOKEN
                '''
            }
        }
    }

    stage('SCA Analysis Using Snyk') {
        steps {
            withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                sh '''
                    export SNYK_TOKEN=$SNYK_TOKEN
                    mvn snyk:test -fn
                '''
            }
        }
    }

    stage('Build Docker Image') {
        steps {
            script {
                app = docker.build("${IMAGE_NAME}")
            }
        }
    }

    stage('Push Docker Image to ECR') {
        steps {
            script {
                docker.withRegistry(
                    'https://079819354494.dkr.ecr.us-west-2.amazonaws.com',
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

                    echo "Deploying Application"

                    kubectl apply -f deployment.yaml -n devsecops

                    echo "Waiting for Rollout"

                    kubectl rollout status deployment/mayank -n devsecops --timeout=300s

                    echo "Pods"
                    kubectl get pods -n devsecops

                    echo "Services"
                    kubectl get svc -n devsecops
                '''
            }
        }
    }

    stage('Wait for Application') {
        steps {
            sh '''
                sleep 180
                echo "Application deployed successfully"
            '''
        }
    }

    stage('Run DAST Using ZAP') {
        steps {
            withKubeConfig([credentialsId: 'kubelogin']) {
                sh '''
                    APP_URL=http://$(kubectl get svc mayank \
                    -n devsecops \
                    -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")

                    echo "Scanning ${APP_URL}"

                    zap.sh \
                    -cmd \
                    -quickurl ${APP_URL} \
                    -quickprogress \
                    -quickout ${WORKSPACE}/zap_report.html
                '''

                archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
            }
        }
    }
}

post {
    success {
        echo 'Pipeline completed successfully'
    }

    failure {
        echo 'Pipeline failed'
    }

    always {
        cleanWs()
    }
}
```

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
