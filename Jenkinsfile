pipeline {
  agent any

  tools { nodejs "NodeJS" }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Archive Artifacts') {
      steps {
        archiveArtifacts artifacts: 'index.html, styles.css, package.json', fingerprint: true
      }
    }
  }
}