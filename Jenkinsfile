pipeline{
    agent any
    tools{
        jdk "jdk17"
        nodejs "node18"
    }
    environment{
        SCANNER_HOME=tool "sonar-scanner"
    }
    stages{
        stage("clean-workspace"){
            steps{
                cleanWs()
            }
        }
        stage("checkout from git"){
            steps{
                git branch: 'main', url: 'https://github.com/Rahul-Mohadikar/Uptime-kuma.git'
            }
        }
        stage("install dependencies"){
            steps{
                sh "npm install"
            }
        }
        stage("soanrqube-analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=uptime \
                    -Dsonar.projectKey=uptime '''
                }
            }
        }
        stage("quality gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage("OWASP-FS-SCAN"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
       stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . &gt; trivyfs.json"
            }
        }
        stage("docker build & push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh "docker build -t uptime ."
                    sh "docker tag uptime rahul70/uptime:latest "
                    sh "docker push rahul70/uptime:latest "
}
                }
            }
        }
        stage("trivy"){
            steps{
                sh "trivy image rahul70/uptime:latest > trivyimage.json"
            }
        }
        stage("remove container"){
            steps{
                sh "docker stop uptime | true"
                sh "docker rm uptime | true"
            }
        }
        stage("deploy to container"){
            steps{
                sh "docker run -d --name uptime -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 rahul70/uptime:latest"
            }
        }
    }
}
