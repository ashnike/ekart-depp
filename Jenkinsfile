pipeline {
    agent any

    tools { 
        maven 'maven3'
        jdk 'jdk-11'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ashnike/Ekart-deployment.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Testing') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: 'sonar-scanner'
                    withSonarQubeEnv('sonarqube') {
                        withCredentials([string(credentialsId: 'sonarq', variable: 'SONAR_TOKEN')]) {
                            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                                -Dsonar.login=${SONAR_TOKEN} -Dsonar.java.binaries=target/classes"
                        }
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DependDC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Building') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'mavenn', jdk: 'jdk-11', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
    }
}
