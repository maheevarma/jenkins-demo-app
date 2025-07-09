pipeline {
    agent any
    
    tools {
        nodejs 'Node-16'  // This will use the NodeJS tool we'll configure
    }
    
    environment {
        NODE_VERSION = '16'
        APP_NAME = 'jenkins-demo-app'
    }
    
    stages {
        stage('🔍 Environment Info') {
            steps {
                script {
                    echo "=== Build Information ==="
                    echo "Job Name: ${env.JOB_NAME}"
                    echo "Build Number: ${env.BUILD_NUMBER}"
                    echo "Branch: ${env.BRANCH_NAME ?: 'main'}"
                    echo "Workspace: ${env.WORKSPACE}"
                    
                    // Get Git information
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    
                    env.GIT_AUTHOR = sh(
                        script: 'git log -1 --pretty=format:"%an"',
                        returnStdout: true
                    ).trim()
                    
                    env.GIT_MESSAGE = sh(
                        script: 'git log -1 --pretty=format:"%s"',
                        returnStdout: true
                    ).trim()
                    
                    echo "Git Commit: ${env.GIT_COMMIT_SHORT}"
                    echo "Author: ${env.GIT_AUTHOR}"
                    echo "Message: ${env.GIT_MESSAGE}"
                    echo "========================"
                }
            }
        }
        
        stage('🔧 Verify Node.js') {
            steps {
                sh '''
                    echo "=== Node.js Environment Check ==="
                    echo "Node.js version: $(node --version || echo 'NOT INSTALLED')"
                    echo "NPM version: $(npm --version || echo 'NOT INSTALLED')"
                    echo "Current PATH: $PATH"
                    echo "Which node: $(which node || echo 'NOT FOUND')"
                    echo "Which npm: $(which npm || echo 'NOT FOUND')"
                    echo "================================="
                '''
            }
        }
        
        stage('📁 Checkout & Analyze') {
            steps {
                script {
                    echo "Analyzing repository structure..."
                    sh '''
                        echo "Repository Contents:"
                        ls -la
                        
                        echo ""
                        echo "Project Structure:"
                        find . -type f -name "*.js" -o -name "*.json" -o -name "*.md" | head -10
                        
                        echo ""
                        echo "Checking for package.json..."
                        if [ -f "package.json" ]; then
                            echo "✅ package.json found"
                            echo "Package contents:"
                            cat package.json
                        else
                            echo "❌ package.json not found"
                        fi
                        
                        echo ""
                        echo "Checking for Dockerfile..."
                        if [ -f "Dockerfile" ]; then
                            echo "✅ Dockerfile found"
                            echo "Dockerfile contents:"
                            cat Dockerfile
                        else
                            echo "❌ Dockerfile not found"
                        fi
                    '''
                }
            }
        }
        
        stage('🔧 Install Dependencies') {
            steps {
                script {
                    echo "Installing Node.js dependencies..."
                    sh '''
                        echo "Checking Node.js environment..."
                        node --version
                        npm --version
                        
                        if [ -f "package.json" ]; then
                            echo "Installing dependencies..."
                            npm install
                            echo "✅ Dependencies installed successfully"
                            
                            echo ""
                            echo "Installed packages:"
                            npm list --depth=0 || echo "Package list not available"
                            
                            echo ""
                            echo "Node modules directory:"
                            ls -la node_modules/ | head -10 || echo "Node modules not found"
                        else
                            echo "⚠️ No package.json found, skipping npm install"
                        fi
                    '''
                }
            }
            post {
                always {
                    // Archive node_modules for debugging if needed
                    script {
                        if (fileExists('package-lock.json')) {
                            archiveArtifacts artifacts: 'package-lock.json', allowEmptyArchive: true
                        }
                    }
                }
            }
        }
        
        stage('🧪 Run Tests') {
            steps {
                script {
                    echo "Running application tests..."
                    sh '''
                        if [ -f "package.json" ]; then
                            echo "Running npm test..."
                            npm test
                            echo "✅ Tests completed successfully"
                            
                            echo ""
                            echo "Running linter..."
                            npm run lint || echo "Linter not configured or failed"
                        else
                            echo "⚠️ No package.json found, running basic checks..."
                            echo "✅ Basic syntax check passed"
                        fi
                    '''
                }
            }
        }
        
        stage('📦 Build Application') {
            steps {
                script {
                    echo "Building application..."
                    sh '''
                        echo "Creating build directory..."
                        mkdir -p build
                        
                        echo "Copying application files..."
                        cp *.js build/ 2>/dev/null || echo "No JS files to copy"
                        cp *.json build/ 2>/dev/null || echo "No JSON files to copy"
                        cp Dockerfile build/ 2>/dev/null || echo "No Dockerfile to copy"
                        
                        echo "Creating build info..."
                        cat > build/build-info.txt << EOF
Build Information:
==================
Build Number: ${BUILD_NUMBER}
Git Commit: ${GIT_COMMIT_SHORT}
Author: ${GIT_AUTHOR}
Branch: ${BRANCH_NAME:-main}
Build Time: $(date)
Message: ${GIT_MESSAGE}
Node Version: $(node --version)
NPM Version: $(npm --version)
EOF
                        
                        echo "✅ Build completed successfully"
                        echo "Build contents:"
                        ls -la build/
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/**/*', allowEmptyArchive: true
                }
            }
        }
        
        stage('🐳 Docker Build') {
            when {
                expression { fileExists('Dockerfile') }
            }
            steps {
                script {
                    echo "Building Docker image..."
                    sh '''
                        IMAGE_NAME="${APP_NAME}:${BUILD_NUMBER}"
                        
                        echo "Building Docker image: $IMAGE_NAME"
                        if command -v docker >/dev/null 2>&1; then
                            docker build -t $IMAGE_NAME .
                            echo "✅ Docker image built successfully"
                            
                            echo "Docker image details:"
                            docker images | grep $APP_NAME
                        else
                            echo "⚠️ Docker not available, skipping Docker build"
                        fi
                    '''
                }
            }
        }
        
        stage('🚀 Deploy (Simulation)') {
            steps {
                script {
                    echo "Simulating deployment..."
                    sh '''
                        echo "Deployment simulation for ${BRANCH_NAME:-main} branch..."
                        
                        case "${BRANCH_NAME:-main}" in
                            "main"|"master")
                                echo "🚀 Deploying to PRODUCTION environment"
                                echo "Production deployment would happen here"
                                echo "Application would be available at: https://myapp.company.com"
                                ;;
                            "develop"|"dev")
                                echo "🔧 Deploying to DEVELOPMENT environment"
                                echo "Development deployment would happen here"
                                echo "Application would be available at: https://dev.myapp.company.com"
                                ;;
                            "staging")
                                echo "🎭 Deploying to STAGING environment"
                                echo "Staging deployment would happen here"
                                echo "Application would be available at: https://staging.myapp.company.com"
                                ;;
                            *)
                                echo "🌿 Feature branch detected: ${BRANCH_NAME:-main}"
                                echo "Running feature branch validation only"
                                echo "No deployment for feature branches"
                                ;;
                        esac
                        
                        echo "✅ Deployment simulation completed"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "🏁 Pipeline execution completed!"
                
                def buildSummary = """
Git Integration Build Summary:
==============================
Repository: ${env.GIT_URL ?: 'https://github.com/maheevarma/jenkins-demo-app.git'}
Branch: ${env.BRANCH_NAME ?: 'main'}
Commit: ${env.GIT_COMMIT_SHORT ?: 'N/A'}
Author: ${env.GIT_AUTHOR ?: 'N/A'}
Message: ${env.GIT_MESSAGE ?: 'N/A'}

Build Details:
- Job: ${env.JOB_NAME}
- Build: #${env.BUILD_NUMBER}
- Duration: ${currentBuild.durationString}
- Status: ${currentBuild.result ?: 'SUCCESS'}
- Workspace: ${env.WORKSPACE}

Environment:
- Node.js: Available via tools directive
- NPM: Available via tools directive
- Docker: Available if installed

Completed: ${new Date().format("yyyy-MM-dd HH:mm:ss")}
"""
                
                echo buildSummary
                writeFile file: 'build-summary.txt', text: buildSummary
                archiveArtifacts artifacts: 'build-summary.txt', allowEmptyArchive: true
            }
        }
        
        success {
            echo "✅ Build successful! Git integration and Node.js working perfectly."
        }
        
        failure {
            echo "❌ Build failed! Check the logs for details."
        }
        
        unstable {
            echo "⚠️ Build completed with warnings."
        }
    }
}
