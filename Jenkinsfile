pipeline {
    agent {
        docker { image 'mcr.microsoft.com/dotnet/sdk:8.0' }
    }
    environment {
        IMAGE_NAME = 'dotnet-sample'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<your-sample-dotnet-repo>.git'
            }
        }
        stage('SAST - Dependency Check') {
            steps {
                sh '''
                    docker run --rm -v $(pwd):/src owasp/dependency-check \
                    --project "dotnet-app" --scan /src --format HTML --out /src/dependency-check-report
                '''
            }
        }
        stage('Build') {
            steps {
                sh 'dotnet build'
            }
        }
        stage('Docker Build & Run') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                    docker run -d -p 8081:80 --name dotnetapp $IMAGE_NAME
                '''
            }
        }
        stage('DAST - ZAP Scan') {
            steps {
                sh '''
                    docker run -t owasp/zap2docker-stable zap-baseline.py -t http://host.docker.internal:8081 -r zap_report.html
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/*.html', allowEmptyArchive: true
        }
    }
}
