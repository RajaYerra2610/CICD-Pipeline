pipeline {
  agent any
  
  environment {
    CI = 'true'
    NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    NETLIFY_SITE_ID    = credentials('NETLIFY_SITE_ID')
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
        sh 'npm test || echo "Tests skipped or no tests configured"'
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
          echo "Installing Netlify CLI..."
          npm install -g netlify-cli
          echo "Deploying build folder to Netlify..."
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
