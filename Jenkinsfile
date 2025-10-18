pipeline {
  agent {
    docker { image 'node:18'; args '-u root:root' }
  }

  environment {
    CI = 'true'
    GIT_EMAIL = 'ci-bot@example.com'
    GIT_NAME  = 'ci-bot'
    NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    NETLIFY_SITE_ID    = credentials('NETLIFY_SITE_ID')
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
        sh 'npm ci'
      }
    }

    stage('Lint') {
      steps {
        sh 'npm run lint || true'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
      post {
        always {
          junit '**/test-results/*.xml'
        }
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

    stage('Deploy to Netlify') {
      when {
        branch 'main'
      }
      steps {
        sh '''
          npm install -g netlify-cli
          echo "Deploying to Netlify..."
          netlify deploy --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --prod
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Build and deployment to Netlify succeeded."
    }
    failure {
      echo "❌ Build failed. Check console output."
    }
  }
}
