pipeline {
    agent any
    
    environment {
        DEV_SCM_REPOSITORY = 'https://github.com/pavanpandu-aws/building-node-react-app.git'
        DEV_SCM_BRANCH = 'master'
        SONAR_PROJECT_KEY = 'a7fe1cf77dea3a840ac7b1c14266211ffa469399'
        SONAR_LOGIN = credentials('sonar-cred')
        SONAR_SERVER_URL = 'http://172.174.215.47:9000/'
        EMAIL_NOTIFICATION = 'pavankuma239@gmail.com'
	JAVA_TOOL = 'openjdk'
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
                    def javaHome = tool JAVA_TOOL
                    env.PATH = "${javaHome}/bin:${env.PATH}"
                    withSonarQubeEnv('SonarQubeDev') {
                        def scannerHome = tool 'sonarScanner'
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} -Dsonar.login=${env.SONAR_LOGIN} -Dsonar.host.url=${env.SONAR_SERVER_URL}"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Quality Gate did not pass. Check SonarQube dashboard for details."
                        }
                    }
                }
            }
        }

        stage("Test") {
            steps {
                echo "Running tests..."
                sh './jenkins/scripts/test.sh'
            }
        }

        stage("Update Version") {
            steps {
                echo "Updating version..."
                script {
                    def packageJson = readJSON file: 'package.json'
                    def currentVersion = packageJson.version
                    def newVersion = currentVersion.replaceAll(/(\d+)$/) { it.toInteger() + 1 }
                    packageJson.version = newVersion
                    writeJSON file: 'package.json', json: packageJson
                    echo "Updated version to: ${newVersion}"
                }
            }
        }

        stage("Commit and Push Version Update") {
            steps {
                echo "Committing and pushing version update..."
                script {
                    sh 'git config user.email "pavankuma239@gmail.com"'
                    sh 'git config user.name "pavan"'
                    sh 'git add package.json'
                    sh 'git commit -m "Bump version"'
                    sh 'git push origin ${env.DEV_SCM_BRANCH}'
                }
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
