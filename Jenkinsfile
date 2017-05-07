pipeline {

	agent any
    
    options {
    	buildDiscarder(logRotator(numToKeepStr:'10'))   	// Keep only the 10 most recent builds 
  	}
  	
  	environment {
  		SONAR = credentials('sonar')						// Sonar Credentials
  		DOCKERHUB = credentials('dockerhub')				// Docker Hub Credentials
		DOCKERHUB_REPO = "craigcloudbees"					// Repo on Docker Hub to push our image to
		DOCKER_IMG_NAME = "sample-rest-server:0.0.1"		// Name of our Docker image
  	}

	stages {
	
		stage('Build') {
			steps {
				sh 'mvn clean package'
				junit allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml'
				
				sh 'pwd'
				sh 'ls -l'
				sh 'ls -l target'
			}
		}
		
		stage('Create Site') {
			when {
				branch 'master'
			}
			steps {
				sh 'mvn site:site'
			}
		}
		
		stage('Create Docker Image') {
			steps {
				// Build Docker image, log into Docker Hub, and push the image to our repo
				sh """
					docker build -t ${DOCKERHUB_REPO}/${DOCKER_IMG_NAME} ./
					docker login -u $DOCKERHUB_USR -p $DOCKERHUB_PSW
					docker push ${DOCKERHUB_REPO}/${DOCKER_IMG_NAME}
				"""
			}
		}
		
		stage('Quality Analysis') {
			when {
				branch 'master'
			}
			steps {
				parallel (
					"Integration Test" : {
						echo 'Run integration tests here...'
					},
					"Sonar Scan" : {
						sh "mvn sonar:sonar -Dsonar.host.url=http://sonar.beedemo.net:9000 -Dsonar.organization=$SONAR_USR -Dsonar.login=$SONAR_PSW"
					}, failFast: true
				)	
			}
		}
		
		stage('Test Docker Image') {
			steps {
				// Run the Docker image we created previously
				sh 'docker run -d -p 4567:4567 ${DOCKERHUB_REPO}/${DOCKER_IMG_NAME}'
				
				// 
				def response = httpRequest 'http://localhost:4567/hello'
				println("Status: "+response.status)
        		println("Content: "+response.content)

				// Stop the Docker image
				sh 'docker stop $(docker ps -q --filter ancestor="${DOCKERHUB_REPO}/${DOCKER_IMG_NAME}") || true'
			}
		}
		
		
    }
    
    post {
    
    	always {
    		echo 'Always'
    	}
    	
    	success {
    		echo 'success'
    	}
    	
    	failure {
    		echo 'failure'
    	}
    }

}