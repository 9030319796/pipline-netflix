pipeline{
    agent {
        label 'linux_slave_0.181'
    }
    tools{
        jdk 'JAVA_HOME'
        nodejs 'nodejs_home'
        
    }
    environment {
        SCANNER_HOME=tool 'sonar-server'
    }
    stages {
        stage('Workspace Cleaning'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/9030319796/pipline-netflix.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix \
                    '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP DP SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Image Build"){
            steps{
                script{
                    
                    sh "docker build --build-arg TMDB_V3_API_KEY=63cc6c7d94ca64ee08a360658a5dc5e4 -t 9030319796/netflix-app ."
                    }
                }
            }
        }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name netflix 9030319796/netflix-app && sleep 10 && docker stop netflix'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubpass')]) {
                    sh 'docker login -u 9030319796 --password ${DockerHubpass}' }
                    sh 'docker push 9030319796/netflix-app:latest'
                }
            }
        }   
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image 9030319796/netflix-app:latest > trivyimage.txt" 
            }
        }
        // stage('Deploy to Kubernetes'){
        //     steps{
        //         script{
        //             dir('Kubernetes') {
        //                 withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                         sh 'kubectl apply -f deployment.yml'
        //                         sh 'kubectl apply -f service.yml'
        //                         sh 'kubectl get svc'
        //                         sh 'kubectl get all'
        //                 }   
        //             }
        //         }
        //     }
        // }
    }
    post {
     sh '''
     Build is successful
     '''
    }
}
