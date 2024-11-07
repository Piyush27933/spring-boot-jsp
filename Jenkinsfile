pipeline {
    agent any

    tools {
        maven 'v3.8.2'
    }
    parameters {
        booleanParam(name: 'PROD_BUILD', defaultValue: true, description: 'Enable this as a production build')
       // string(name: 'SERVER_IP', defaultValue: '127.0.0.1', description: 'Provide production server IP Address.')
    }

    environment {
        NEXUS_URL = 'http://<Nexus-IP>:8081/repository/maven-releases/' // Replace with your Nexus server URL
        NEXUS_CREDENTIALS_ID = 'nexus-creds' // Credentials ID for Nexus
        SONARQUBE_SERVER = 'SonarQube' // Define SonarQube server in Jenkins configuration
        VERSION = "1.0.${env.BUILD_NUMBER}"
    }

    stages {
        stage('Source') {
            steps {
                git branch: 'Piyush', changelog: false, credentialsId: 'github-token1', poll: false, url: 'https://github.com/Piyush27933/spring-boot-jsp.git'
            }
        }
        
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SONARQUBE_SERVER') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Validate') {
            steps {
                sh 'mvn validate'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Test') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Test') {
                    steps {
                        echo 'Doing integration test' // Placeholder for integration testing steps
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                script {
                    def nexusPath = "${NEXUS_URL}your-group-id/news-${VERSION}.jar"
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_URL}",
                        groupId: 'your-group-id',
                        version: VERSION,
                        repository: 'maven-releases',
                        credentialsId: NEXUS_CREDENTIALS_ID,
                        artifacts: [
                            [artifactId: 'news', classifier: '', file: "target/news-${VERSION}.jar", type: 'jar']
                        ]
                    )
                    echo "Artifact published to Nexus at ${nexusPath}"
                }
            }
        }

        stage('Deploying to EC2') {
            when {
                expression { return params.PROD_BUILD }
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deployment-key', keyFileVariable: 'SSHKEY', usernameVariable: 'SSHUSER')]) {
                    sh '''
                        version=$(perl -nle 'print "$2" if /<(version>)(v(\\d\\.){2}\\d)<\\/\\1/' pom.xml)
                        rsync -avzP -e "ssh -o StrictHostKeyChecking=no -i ${SSHKEY}" target/news-${version}.jar ${SSHUSER}@${SERVER_IP}:/home/headless-newsapp/newsapp/
                        ssh -o StrictHostKeyChecking=no -i ${SSHKEY} ${SSHUSER}@${SERVER_IP} sudo /usr/bin/systemctl restart newsapp.service
                    '''
                }
            }
        }
    }

    
}
