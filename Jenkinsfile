pipeline {
  agent any
  
  tools {
    nodejs "node"  // must match NodeJS installation name in Jenkins
  }
  
  environment {
    CI = 'false'  // Disable CI mode to prevent treating warnings as errors
    NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    NETLIFY_SITE_ID = '382119c6-07f6-458f-a8ea-487fa3669841'
  }
  
  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 30, unit: 'MINUTES')
  }
  
  stages {
    stage('Checkout') {
      steps {
        echo '📥 Checking out source code...'
        checkout scm
      }
    }
    
    stage('Install Dependencies') {
      steps {
        echo '📦 Installing dependencies...'
        sh 'npm ci --prefer-offline --no-audit'
      }
    }
    
    stage('Build') {
      steps {
        echo '🏗️ Building project...'
        sh 'npm run build'
      }
      post {
        success {
          echo '✅ Build artifacts created successfully'
          archiveArtifacts artifacts: 'build/**/*', fingerprint: true, allowEmptyArchive: false
        }
      }
    }
    
    stage('Verify Build Output') {
      steps {
        echo '🔍 Verifying build output...'
        sh '''
          if [ ! -d "build" ]; then
            echo "Error: build directory not found!"
            exit 1
          fi
          echo "Build directory contents:"
          ls -la build/
        '''
      }
    }
    
    stage('Deploy to Netlify') {
      when {
        branch 'main'
      }
      steps {
        echo '🚀 Deploying to Netlify...'
        sh '''
          # Install netlify-cli if not already available
          npm install -g netlify-cli@latest
          
          # Verify authentication
          echo "Testing Netlify authentication..."
          netlify status --auth=$NETLIFY_AUTH_TOKEN || {
            echo "❌ Netlify authentication failed!"
            echo "Please verify your NETLIFY_AUTH_TOKEN credential in Jenkins"
            exit 1
          }
          
          # Deploy to production
          echo "Deploying to site: $NETLIFY_SITE_ID"
          netlify deploy \
            --dir=build \
            --site=$NETLIFY_SITE_ID \
            --auth=$NETLIFY_AUTH_TOKEN \
            --prod \
            --message="Jenkins build #${BUILD_NUMBER}"
        '''
      }
      post {
        success {
          echo '✅ Deployment successful!'
          echo "🌐 Visit: https://app.netlify.com/sites/${env.NETLIFY_SITE_ID}"
        }
        failure {
          echo '❌ Deployment failed - check Netlify credentials and site access'
        }
      }
    }
  }
  
  post {
    success {
      echo "✅ Pipeline completed successfully!"
      echo "Build #${BUILD_NUMBER} deployed to production"
    }
    failure {
      echo "❌ Pipeline failed at stage: ${env.STAGE_NAME}"
      echo "Check the logs above for details"
    }
    always {
      echo "Pipeline execution time: ${currentBuild.durationString}"
    }
  }
}
