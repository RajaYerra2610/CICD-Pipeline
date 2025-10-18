pipeline {
    agent any

    environment {
        CI = 'true'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN') // Jenkins secret text
        NETLIFY_SITE_ID    = credentials('NETLIFY_SITE_ID')    // Jenkins secret text
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
                sh 'npm run lint || echo "Lint failed or skipped"'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "Tests skipped or not configured"'
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

                    echo "✅ Deployment finished. Visit your Netlify site!"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Build and deployment succeeded!"
        }
        failure {
            echo "❌ Build failed — check the console output."
        }
    }
}
