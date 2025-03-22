def registry = 'https://firstquad.jfrog.io/'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    DOCKER_TAG = '2.1.3'
    DOCKER_IMAGE_NAME = 't-trend'
    JFROG_REGISTRY = 'https://firstquad.jfrog.io'
    ARTIFACTORY_REPO = 'firstquad.jfrog.io/fqts-docker-docker-local'
}
    stages {
        stage("build") {
            steps {
                // git branch: 'main', url: 'https://github.com/Prathameshd2799/tweet-trend-for-Project2.git' 
                echo "build started"
                sh 'mvn clean deploy'
                echo "build completed"
            }
            
        }
            stage('SonarQube analysis') {
             environment {
             scannerHome = tool 'FQTS-sonar-scanner'
                   }
            steps{
                withSonarQubeEnv('FQTS-sonarqube-server') { 
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
            stage("Jar Publish") {
            steps {
                 script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jgrog-creds"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "jenkins-jfrog-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
                   }
            }

        }
             stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} .
                    """
                }
            }   
        } 
        stage('Publish Docker Image to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jgrog-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    echo $PASSWORD | docker login $JFROG_REGISTRY --username $USERNAME --password-stdin
                    echo '<--------------- docker login done --------------->' 
                    docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${ARTIFACTORY_REPO}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    docker push ${ARTIFACTORY_REPO}/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                    }
                }
            }
        }   
        stage('deploy'){
            steps {
                script{
                    sh """
                    chmod 777 deploy.sh
                    ./deploy.sh
                    """
                }
            }
        }   
    }
}


