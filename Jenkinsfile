pipeline {
    agent any
    stages {
        stage('Initialize') {
            steps {
                echo "Esta es el inicio"
            }
        }
        stage('Build') {
            steps {
                echo "Este es el build"
            }
        }
        stage('Test') {
            steps {
                echo "Este es el Test"
            }
        }
        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQube Conexion'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'SonarGrupo3', installationName: 'SonarQube123') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \\
                    -Dsonar.projectKey=modulo3 \\
                    -Dsonar.projectName=modulo3 \\
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
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: '192.168.100.13:8081',
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: 'grupo3-hosted',
                            credentialsId: 'nexus',
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
}
