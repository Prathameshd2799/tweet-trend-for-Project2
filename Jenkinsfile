def registry = 'https://fqts01pd.jfrog.io'

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
        
        stage("Jar Publish") {
            steps {
                script {
                    echo "<--------------- Jar Publish Started --------------->"
                    
                    def server = Artifactory.newServer(
                        url: "${registry}/artifactory", 
                        credentialsId: "jenkins-jforg-creds"
                    )
                    
                    def properties = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"
                    
                    def uploadSpec = """
                    {
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "fqts-01pd-libs-release-local/{1}",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }
                    """
                    
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    
                    echo "<--------------- Jar Publish Ended --------------->"
                }
            }
        }
    }
}
