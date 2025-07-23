pipeline {
	agent any
	 tools {
		nodejs 'nodejs23'
	}
	environment {
		SCANNER_HOME = tool 'sonar-scanner'
	}
	stages {
		stage('Git Checkout'){
			steps {
				git url : 'https://github.com/nagarajudev8/3-Tier-DevSecOps-Mega-Project.git' , branch: 'dev-docker-deploy'
			}
		}
		stage('Front End Compile'){
			steps{
				dir ('client'){
					echo 'FrontEnd Compilation Started'
					sh 'find . -name "*.js" -exec node --check {} +'
					echo 'FrontEnd Compilation Completed'
				}
			}
		}
		stage('BackEnd Compile') {
			steps {
				dir('api'){
					echo 'Backend compilation Started'
					sh 'find . -name "*.js" -exec node --check {} +'
					echo 'Backend Compilation Completed'
				}
			}
		}
		stage('GitLeaks Scan'){
			steps {
				echo 'GitLeaks Scan Started'
				sh 'gitleaks detect --source ./client --exit-code 1'
				sh 'gitleaks detect --source ./api --exit-code 1'
				echo 'GitLeaks Scan Completed'
			}
		}
		stage('SonarQube Analysis'){
			steps{
				echo 'SonarQube Analysis started'
				withSonarQubeEnv('sonar'){
					sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
							-Dsonar.projectKey=NodeJS-Project '''
				}
				echo 'SonarQube Analysis Completed'
			}
		}
		stage('Quality Gate Check'){
			steps{
				echo 'Quality Gate Check started'
				timeout(time:2 , unit: 'MINUTES'){
					waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
				}
				echo "Quality Gate Check completed"
			}
		}
		stage('Trivy FS Scan'){
			steps {
				echo 'Trivy FS SCAN STARTED'
				sh 'trivy fs  --format table -o fs-scan-report.html .'
				echo 'Trivy FS SCAN COMPLETED'
			}
		}
		stage('FrontEnd compile build push to Docker'){
			steps{
				script{
					echo 'FrontEnd Build push started'
					dir('client'){
						sh 'docker build -t naga8/frontend:latest .'
						sh 'trivy image --format table -o frontend-image-scan-report.html naga8/frontend:latest'
						withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
							sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
							sh 'docker push naga8/frontend:latest'
						}
						sh 'docker push naga8/frontend:latest'
					}
					echo 'Frontend Build push completed'
				}
			}
		}
		stage('BackEnd Compile build push to Docker'){
			steps{
				script{
					echo 'BackEnd Build Push started'
					dir('api'){
						sh 'docker build -t naga8/backend:latest .'
						sh 'trivy image --format table -o backend-image-scan-report.html naga8/backend:latest'
						withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
							sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
							sh 'docker push naga8/backend:latest'
						}
						sh 'docker push naga8/backend:latest'
					}
					echo 'Backend Build Push Completed'
				}
			}
		}
		stage('Docker Compose UP'){
			steps{
				echo 'Docker Compose started'
				sh 'docker compose up -d'
				echo 'Docker compose Completed'
			}
		}
	}
}
			
						
			
				
			
		
				
						
