pipeline {
    agent any
    stages {
        stage('Limpieza y construccion del paquete') {
            steps {
                //echo "Este es el build"
                sh "mvn clean package"
            }
        }
        stage('Revisión del código') {
            steps {
                echo "Teste ejecutado correctamente"
            }
        }
        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQube'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'sonar', installationName: 'SonarQube') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \\
                    -Dsonar.projectKey=tarea \\
                    -Dsonar.projectName=Tarea3 \\
                    -Dsonar.sources=./ \\
                    -Dsonar.java.binaries=target/classes/ """
                    
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: 'pom.xml'
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    //echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: '192.168.1.203:8081',
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: 'Modulo3-hosted',
                            credentialsId: 'MavenNexus',
                            artifacts: [
                                [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging]
                            ]
                        )
                    } else {
                        error "* File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
    post {
        always{
            slackSend(channel: '#modulo-3-actividad-grupal', message: 'Se inició la ejecución :rocket: del PIPELINE - Grupo3')
        }
        failure{
            slackSend(channel: '#modulo-3-actividad-grupal', message: ':warning: Falló la ejecución del PIPELINE - Grupo3 :warning:')
        }
        success{
            slackSend(channel: '#modulo-3-actividad-grupal', message: 'La ejecución del PIPELINE fue exitosa - Grupo3  :smile:')
        }
    }
}
