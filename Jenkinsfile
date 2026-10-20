pipeline {
  agent { label 'docker-azure-cli' }

  environment {
    REGISTRY   = 'demoapp1234-gve4d0a6huhxchgu.azurecr.io'
    IMAGE      = 'day1-demo'
    IMAGE_NAME = "${REGISTRY}/${IMAGE}:${BUILD_NUMBER}"
    JAVA_HOME  = '/usr/lib/jvm/java-17-openjdk-amd64'
    PATH       = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Test') {
      steps { sh 'mvn -B clean test' }
    }

    stage('Package') {
      steps { sh 'mvn -B package -DskipTests' }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t "$IMAGE_NAME" .'
      }
    }

    stage('Security Scan') {
      steps {
        sh '''
          mkdir -p security-reports
          trivy image --scanners vuln --format json \
            --output security-reports/trivy-image.json "day1-demo"

          trivy convert --format table --severity HIGH,CRITICAL \
            --exit-code 1 security-reports/trivy-image.json
        '''
      }
    }

    stage('Push to ACR') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'acr-push',
            usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
          sh '''set +x
echo "$ACR_PASS" | docker login "$REGISTRY" -u "$ACR_USER" --password-stdin
docker push "$IMAGE_NAME"
docker logout "$REGISTRY"'''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'target/*.jar, security-reports/*.json',
                       allowEmptyArchive: true
    }
  }
}
