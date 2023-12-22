pipeline {
    agent any
    
    environment {
        DEV_SCM_REPOSITORY = 'https://github.com/pavanpandu-aws/building-node-react-app.git'
        DEV_SCM_BRANCH = 'master'
        SONAR_PROJECT_KEY = 'a7fe1cf77dea3a840ac7b1c14266211ffa469399'
        SONAR_LOGIN = credentials('sonar-cred')
        SONAR_SERVER_URL = 'http://172.174.215.47:9000/'
        EMAIL_NOTIFICATION = 'pavankuma239@gmail.com'
    }
    
    stages {
        stage("Check Node.js and npm") {
            steps {
                script {
                    def nodeInstalled = tool 'NodeJS' == null
                    if (nodeInstalled) {
                        echo "Node.js not found. Installing Node.js..."
                        tool name: 'NodeJS', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                    } else {
                        echo "Node.js is already installed."
                    }

                    def npmHome = sh(script: 'which npm', returnStatus: true).trim()
                    if (npmHome) {
                        echo "npm is already installed."
                    } else {
                        echo "npm not found. Installing npm..."
                        sh 'curl -L https://www.npmjs.com/install.sh | sh'
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
