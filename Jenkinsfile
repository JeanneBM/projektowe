pipeline {
    agent {
        label 'Slave'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'auto_maven'
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
  
    stages {
        stage('Clear running apps') {
           steps {
               // Clear previous instances of app built
               sh 'docker rm -f pandaapp || true'
           }
        }
        stage('Get Code') {
            steps {
                // Get some code from a GitHub repository
                checkout scm
            }
        }
        stage('Build and Junit') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn clean install"
            }
        }
        stage('Build Docker image'){
            steps {
                sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker app') {
            steps {
                sh "docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}"
            }
        }
        stage('Test Selenium') {
            steps {
                sh "mvn test -Pselenium"
            }
        }
        stage('Deploy jar to artifactory') {
            steps {
                configFileProvider([configFile(fileId: '70a33736-937d-4f10-9425-0d7a1ede2339', variable: 'MAVEN_SETTINGS')]) {
                    sh "mvn -gs $MAVEN_SETTINGS deploy -Dmaven.test.skip=true -e"
                }
            } 
            post {
                always { 
                    sh 'docker stop pandaapp'
                    deleteDir()
                }
            }
        }
    }
}
