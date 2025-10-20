pipeline {
  agent any

  tools {
    nodejs "node" // Must match your Jenkins NodeJS installation name
  }

  environment {
    CI = 'true'
    GIT_EMAIL = 'ci-bot@example.com'
    GIT_NAME  = 'ci-bot'
    GITHUB_PAT = credentials('GITHUB_PAT') // Jenkins credential ID
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    ansiColor('xterm') // Optional: colorize console output
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci' // Clean reproducible install
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
      }
      post {
        success {
          archiveArtifacts artifacts: 'build/**', fingerprint: true
        }
      }
    }

    stage('Deploy to GitHub Pages') {
      when {
        branch 'main'
      }
      steps {
        sh '''
          set -e
          cd build
          git init
          git config user.email "$GIT_EMAIL"
          git config user.name "$GIT_NAME"
          git add .
          git commit -m "Deploy from Jenkins: ${GIT_COMMIT}"
          git push --force https://${GITHUB_PAT}@github.com/RajaYerra2610/CICD-Pipeline.git master:gh-pages
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Build & deploy succeeded."
    }
    failure {
      echo "❌ Build failed. Check console output for details."
    }
  }
}
