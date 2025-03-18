pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }

    stages {
        stage("Build") {
            steps {
                script {
                    echo "<--------------- Build Started --------------->"
                    sh 'mvn clean deploy'
                    echo "<--------------- Build Completed --------------->"
                }
            }
        }

        stage("SonarQube Analysis") {
            environment {
                scannerHome = tool 'FQTS-sonar-scanner'
            }
            steps {
                script {
                    echo "<--------------- SonarQube Analysis Started --------------->"
                    withSonarQubeEnv('FQTS-sonarqube-server') { 
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                    echo "<--------------- SonarQube Analysis Completed --------------->"
                }
            }
        }

        stage("Configure Artifactory") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'artifactory-token', variable: 'ARTIFACTORY_TOKEN')]) {
                        def rtServer = Artifactory.newServer(
                            id: 'Artifactory-1',
                            url: 'https://fqts01pd.jfrog.io/artifactory',
                            username: 'prathameshdhalkar188@gmail.com',
                            accessToken: ARTIFACTORY_TOKEN,
                            bypassProxy: true,
                            timeout: 300
                        )
                    }
                }
            }
        }
    }
}
