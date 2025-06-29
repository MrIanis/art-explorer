pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-id')
    SSH_CREDENTIALS      = credentials('vm-ssh-key')
    IMAGE_NAME           = "moncompte/art-explorer"
  }

  stages {
    stage('Checkout') {
      steps { git url: 'https://github.com/o0morgan0o/art-explorer.git' }
    }
    stage('Tests') {
      steps {
        sh 'pip install -r requirements.txt pytest'
        sh 'pytest --maxfail=1 --disable-warnings -q'
      }
    }
    stage('Build Docker') {
      steps { script { docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}") } }
    }
    stage('Push Docker') {
      steps {
        script {
          docker.withRegistry('', 'DOCKERHUB_CREDENTIALS') {
            docker.image("${IMAGE_NAME}:${env.BUILD_NUMBER}").push()
          }
        }
      }
    }
    stage('Deploy sur VM') {
      steps {
        sshagent (credentials: ['SSH_CREDENTIALS']) {
          sh """
            ssh -o StrictHostKeyChecking=no user@ma-vm '
              docker pull ${IMAGE_NAME}:${env.BUILD_NUMBER} &&
              docker rm -f art-explorer || true &&
              docker run -d --name art-explorer -p 80:5000 ${IMAGE_NAME}:${env.BUILD_NUMBER}
            '
          """
        }
      }
    }
  }

  post {
    failure {
      mail to: 'ton.email@domaine.com',
           subject: "Échec build #${env.BUILD_NUMBER}",
           body: "Consulte la console Jenkins pour diagnostics."
    }
  }
}
