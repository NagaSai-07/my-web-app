pipeline {
  agent any
  environment {
    SONAR_TOKEN = credentials('sonarqube') // Jenkins-stored token
    DOCKER_IMAGE = "my-web-app:${BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "sonar-scanner -Dsonar.login=${SONAR_TOKEN}"
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE} ."
      }
    }
    stage('Run Container') {
      steps {
        sh '''
          docker rm -f my-web-app || true
          docker run -d --name my-web-app -p 80:5000 ${DOCKER_IMAGE}
        '''
      }
    }
  }
  post {
    success {
      echo "✅ Deployment successful: http://${env.BUILD_URL.replaceAll('/job.*','')}"
    }
    failure {
      echo "❌ Deployment failed."
    }
  }
}
