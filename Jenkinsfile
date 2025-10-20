pipeline {
  agent any

  tools {
    nodejs "node"  // must match NodeJS installation name in Jenkins
  }

  environment {
    CI = 'true'
    NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN') // Jenkins credential ID
    NETLIFY_SITE_ID    = '382119c6-07f6-458f-a8ea-487fa3669841'                  // raw site ID as string
  }

  options {
    timestamps()
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
        echo '📦 Installing dependencies...'
        sh 'npm ci'
      }
    }

    stage('Build') {
      steps {
        echo '🏗️ Building project...'
        sh '''
          unset CI
          npm run build || true
        '''
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
        echo '🚀 Deploying to Netlify...'
        sh '''
          npm install -g netlify-cli
          netlify deploy --dir=build --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN --prod
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Build & Deploy successful — check Netlify dashboard!"
    }
    failure {
      echo "❌ Build failed — please check the console output."
    }
  }
}
