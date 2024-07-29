pipeline {
    agent {
        docker {
            image 'nexus.creverse.com/devops/build/ubi9/npm:16'
            registryUrl 'https://nexus.creverse.com/'
            registryCredentialsId 'cred_nexus'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        NEXUS = credentials('cred_nexus')
        SQ_PROJECT_KEY = credentials('cred_sonarqube_vue_prj_key')
        NODE_OPTIONS = "--openssl-legacy-provider"
    }

    stages {
        stage('Install') {
            steps {
                withNPM(npmrcConfig: 'npm_group_config') {
                    sh 'npm --version'
                    sh 'npm install'
                }
            }
        }

        stage('test') {
            steps {
                script {
                    try {
                        sh 'npm run test'
                    } catch (Exception e) {
                        echo 'Exception occurred: ' + e.toString()
                    }
                }
            }
        }

        stage('lint') {
            steps {
                script {
                    try {
                        sh 'npm run lint'
                    } catch (Exception e) {
                        echo 'Exception occurred: ' + e.toString()
                    }
                }
            }
        }        

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('sonarqube') {
            steps {
                withSonarQubeEnv(installationName: 'Sonarqube', credentialsId: 'cred_sonarqube_token') {
                    sh 'sonar-scanner -Dsonar.projectKey="$SQ_PROJECT_KEY" -Dsonar.sources=.'
                }                
            }
        }

        stage('publish') {
            steps {
                withNPM(npmrcConfig: 'npm_host_config') {
                    sh 'npm publish'
                }
            }
        }
    }
}
