pipeline {
    agent {
        label 'AGENT-1'
    }
    options { 
        // Timeout counter starts BEFORE agent is allocated
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        def appVersion = ''
        def nexusUrl = 'nexus.puneeth.cloud:8081'
    }
    parameters {
        string(name: 'appVersion', defaultValue: '1.0.0', description: 'What is the application version?')
    }
    stages {
        stage('Read the Version') {
            steps {
                script {
                  def packageJson = readJSON file: 'package.json'
                  appVersion = packageJson.version
                  echo "app version : $appVersion"
                }                
            }
        }
        stage ('Build') {
            steps {
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                ls -ltr 
                """
            }
        }
        stage ('Nexus Artifact Upload') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: 'frontend',
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "frontend",
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage ('Deploy') {
            steps {
                script {
                    def params = [
                string(name: 'appVersion', value: "${appVersion}")
                ]
                    build job: 'frontend-deploy', parameters: params, wait: false
                }                
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success{
            echo 'I will when pipeline is success'
        }
        failure{
            echo 'I will when pipeline is failure'
        }
    }
}
    

