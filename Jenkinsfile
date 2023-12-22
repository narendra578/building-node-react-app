pipeline {
    agent any
    
    environment {
        DEV_SCM_REPOSITORY = 'https://github.com/pavanpandu-aws/building-node-react-app.git'
        DEV_SCM_BRANCH = 'master'
        SONAR_PROJECT_KEY = 'demo'
        SONAR_SERVER_URL = 'http://172.174.215.47:9000'
        SONAR_LOGIN = 'a7fe1cf77dea3a840ac7b1c14266211ffa469399'
        EMAIL_NOTIFICATION = 'pavankuma239@gmail.com'
	JAVA_HOME = '/opt/jdk-21.0.1'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }
    
    stages {
	stage("Check and Install Node.js and npm") {
	    steps {
	        script {
	            // Check if Node.js is installed
	            def nodeInstalled = sh(script: 'command -v node', returnStatus: true) == 0
	
	            if (!nodeInstalled) {
	                echo "Node.js not found. Installing Node.js..."
	                sh 'curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -'
	                sh 'sudo apt-get install -y nodejs'
	            } else {
	                echo "Node.js is already installed."
	            }
	
	            // Check if npm is installed
	            def npmInstalled = sh(script: 'command -v npm', returnStatus: true) == 0
	
	            if (!npmInstalled) {
	                echo "npm not found. Installing npm..."
	                sh 'sudo apt-get install -y npm'
	            } else {
	                echo "npm is already installed."
	            }
	        }
	    }
	}


	stage("Checkout") {
            steps {
                echo "Checking out code..."
                checkout([$class: 'GitSCM', branches: [[name: "${env.DEV_SCM_BRANCH}"]], userRemoteConfigs: [[url: env.DEV_SCM_REPOSITORY]]])
            }
        }

        stage("SonarQube analysis") {
            steps {
                script {
                    // Use the installed Sonar Scanner
                    def scannerHome = tool 'sonarScanner'
                    
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.login=${SONAR_LOGIN} \
                        -Dsonar.host.url=${SONAR_SERVER_URL}
                    """
                }
            }
        }


        stage("Deliver") {
            steps {
                echo "Running tests..."
                sh './jenkins/scripts/deliver.sh'
            }
        }

        stage("Update Version") {
	    steps {
                echo "Updating version..."
                script {
                    // Run the AWK script to update the version
                    sh '''
                        #!/usr/bin/awk

                        awk -F'["]' -v OFS='"'  '/"version":/{split($4,a,".");$4=a[1]+1"."a[2]"."a[3]+1};1' ./package.json > ./package2.json && mv ./package2.json ./package.json
                    '''
                }
            }
	}



        stage("Commit and Push Version Update") {
	    steps {
	        script {
	            // Committing and pushing version update
	            withCredentials([usernamePassword(credentialsId: 'gitcred', passwordVariable: 'GIT_ACCESS_TOKEN', usernameVariable: 'GIT_USERNAME')]) {
	                sh '''
	                    git config --global user.email "pavankuma239@gmail.com"
	                    git config --global user.name "pavan"
	                    git checkout -b master
	                    git add package.json
	                    git commit -m "Bump version"
	                    git push https://${GIT_USERNAME}:${GIT_ACCESS_TOKEN}@github.com/pavanpandu-aws/building-node-react-app.git master
	                '''
	            }
	        }
	    }
	

	
            post {
                failure {
                    echo "Build failed. Sending alerts..."
                    mail to: "${env.EMAIL_NOTIFICATION}",
                         subject: "Jenkins Build Failed: ${currentBuild.fullDisplayName}",
                         body: "Build failed. Check Jenkins console output for details."
                }
            }
        }
    }
}
