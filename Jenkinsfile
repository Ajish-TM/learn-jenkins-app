pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '4aa620d2-75f2-4f50-aa27-380aeedd9a5d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                          sh '''
                    echo "🚀 Installing Netlify CLI..."
                    npm install netlify-cli

                    echo "🔍 Checking Netlify CLI version..."
                    node_modules/.bin/netlify --version

                    echo "🌐 Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status --auth $NETLIFY_AUTH_TOKEN || echo "⚠️  Skipping status check (not authenticated session)"

                    echo "📤 Uploading build directory to Netlify..."
                    node_modules/.bin/netlify deploy --auth $NETLIFY_AUTH_TOKEN --prod --dir=build --message "CI Deploy via Jenkins"

                    echo "✅ Deployment complete."
                '''

            }
        }
    }
}
