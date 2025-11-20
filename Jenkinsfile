pipeline {
    agent any

    triggers { // Sondear repositorio a intervalos regulares
        pollSCM('* * * * *')
    }
    tools {
        maven 'maven_lts'
    }
    stages {
        stage('init') {
            steps {
                echo 'Hola mundo'
                git 'https://github.com/jmagit/demos-devops.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    recordCoverage(tools: [[parser: 'JACOCO']])
                }
            }
        }
        stage('build') {
            when {
                branch 'production' 
            }
            failFast true
            parallel {
                stage('package') {
                    steps {
                        sh 'mvn package -DskipTests'
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, followSymlinks: false
                        }
                    }
                }
                stage('site') {
                    steps {
                        sh 'mvn site'
                    }
                    post {
                        success {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                icon: '', 
                                keepAll: false, 
                                reportDir: 'target/site/', 
                                reportFiles: 'index.html', 
                                reportName: 'Documentacion del sitio', 
                                reportTitles: 'Documentacion del sitio', 
                                useWrapperFileDirectly: true
                                ])
                        }
                    }
                }
            }
        }
        stage('deploy') {
            when {
                branch 'production' 
            }
            steps {
                mail to: 'amd@example.com',
                 subject: "Pendiente de aprobacion el depliege de ${currentBuild.fullDisplayName}",
                 body: "Para la aprobacion entre en ${env.BUILD_URL+'pipeline-overview'}"
                timeout(time: 3, unit: 'MINUTES') {
                    input cancel: 'Cancelar', message: 'Procedo al despliegue o aborto', ok: 'Aceptar'
                }
                sh 'mvn install -DskipTests'
            }
        }
    }
    post {
        success {
            mail to: 'team@example.com',
                 subject: "OK Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something in ${env.BUILD_URL}"
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}
