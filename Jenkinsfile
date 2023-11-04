pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=opidbuggywebapp -Dsonar.organization=opidbuggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=7535163adaa6a6dafa3e4e7df668941b00a0d450'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("opidbuggy")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://284526353281.dkr.ecr.eu-west-1.amazonaws.com/', 'ecr:eu-west-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}

      stage('Kubernetes Deployment of ASG Bugg Web Application') {
	      steps {
	        withKubeConfig([credentialsId: 'kubelogin']) {
		          sh('kubectl delete all --all -n devsecops')
		          sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		        }
	        }
   	}

	    
  }
}
