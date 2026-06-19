pipeline {
agent any

```
tools {
    maven 'Maven_3_8_4'
}

stages {

    stage('Create Namespace') {
        steps {
            withKubeConfig([credentialsId: 'kubelogin']) {
                sh '''
                kubectl get namespace devsecops >/dev/null 2>&1 || \
                kubectl create namespace devsecops
                '''
            }
        }
    }

    stage('Compile and Run Sonar Analysis') {
        steps {
            sh '''
            mvn clean verify sonar:sonar \
            -Dsonar.projectKey=mayanko15 \
            -Dsonar.organization=mayanko15 \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.token=$SONAR_TOKEN
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
            script {
                app = docker.build("mayank")
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
                kubectl delete all --all -n devsecops || true
                kubectl apply -f deployment.yaml -n devsecops

                kubectl get pods -n devsecops
                kubectl get svc -n devsecops
                '''
            }
        }
    }

    stage('Wait for Testing') {
        steps {
            sh '''
            sleep 360
            echo "Application deployed on Kubernetes"
            '''
        }
    }

    stage('Run DAST Using ZAP') {
        steps {
            withKubeConfig([credentialsId: 'kubelogin']) {
                sh '''
                APP_URL=$(kubectl get svc mayank \
                -n devsecops \
                -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")

                echo "Target URL: http://$APP_URL"

                docker run --rm \
                -v ${WORKSPACE}:/zap/wrk \
                ghcr.io/zaproxy/zaproxy:stable \
                zap-baseline.py \
                -t http://$APP_URL \
                -r zap_report.html
                '''
            }

            archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
        }
    }
}

post {
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
// 		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=mayank -Dsonar.organization=mayank -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=5e3f77bafff78138c2427798e42b4c1232da9502'
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
//                  app =  docker.build("mayank")
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
// 				sh('zap.sh -cmd -quickurl http://$(kubectl get services/mayank --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
// 				archiveArtifacts artifacts: 'zap_report.html'

// 		    }
// 	     }
//        } 
//   }
// }
