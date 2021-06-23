pipeline {
    agent {
      label 'Slave'
    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "m3"
    }
    
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }

    stages {
        
        stage('Clear running apps') {
            steps {
                // Clear previous instance of app built
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
                // Get some code from a GitHub repository
                sh 'mvn clean install'
            }
        }
        
         stage('Build Docker image'){
             steps {
                // Build docker image
                sh 'mvn package -Pdocker'
             }
         }
         
         stage('Run Docker app') {
             steps {
                 // Run docker app from defined 
                sh 'docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}'
             }
         }

        stage('Test Selenium') {
            steps {
                // Get some code from a GitHub repository
                sh 'mvn test -Pselenium'
            }
        }
        
        stage('Deploy jar to Artifactory') {
            steps {
                configFileProvider([configFile(fileId: 'fbb7ec03-d6f7-49b8-984c-e65df188fb97', variable: 'MySettings')]) {
                // some block
                sh "mvn -s $MySettings deploy -Dmaven.test.skip=true -e"
               }
            }
        }
    }
    post {
        always {
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }
}