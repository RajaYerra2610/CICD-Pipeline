pipeline {
  agent {
    docker { image 'node:18'; args '-u root:root' }
  }
  environment {
    CI = 'true'
    GIT_EMAIL = 'ci-bot@example.com'
    GIT_NAME  = 'ci-bot'
    GITHUB_PAT = credentials('GITHUB_PAT')  // Jenkins credential id
  }
  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci'  // reproducible install
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
          git push --force https://${GITHUB_PAT}@github.com/<you>/my-app.git master:gh-pages
        '''
      }
    }
  }

  post {
    success {
      echo "Build & deploy succeeded."
    }
    failure {
      echo "Build failed. Check console output."
    }
  }
}
