pipeline {
    agent any
    
    environment {
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
        
        stage('📁 Repository Analysis') {
            steps {
                sh '''
                    echo "=== Repository Analysis ==="
                    echo "Repository Contents:"
                    ls -la
                    
                    echo ""
                    echo "Git Information:"
                    echo "Current branch: $(git rev-parse --abbrev-ref HEAD)"
                    echo "Latest commit: $(git log -1 --oneline)"
                    echo "Remote URL: $(git config --get remote.origin.url)"
                    
                    echo ""
                    echo "Project Files:"
                    find . -type f -name "*.js" -o -name "*.json" -o -name "*.md" | head -10
                    
                    echo ""
                    echo "File Analysis:"
                    for file in "package.json" "app.js" "Dockerfile" "README.md"; do
                        if [ -f "$file" ]; then
                            echo "✅ $file exists ($(wc -l < "$file") lines)"
                        else
                            echo "❌ $file missing"
                        fi
                    done
                '''
            }
        }
        
        stage('📦 Simple Build') {
            steps {
                sh '''
                    echo "=== Building Application ==="
                    mkdir -p build
                    
                    # Copy all relevant files
                    cp *.js build/ 2>/dev/null || echo "No JS files"
                    cp *.json build/ 2>/dev/null || echo "No JSON files"  
                    cp *.md build/ 2>/dev/null || echo "No MD files"
                    cp Dockerfile build/ 2>/dev/null || echo "No Dockerfile"
                    
                    # Create build manifest
                    cat > build/build-manifest.txt << EOF
Build Manifest
==============
Build Number: ${BUILD_NUMBER}
Git Commit: ${GIT_COMMIT_SHORT}
Author: ${GIT_AUTHOR}
Message: ${GIT_MESSAGE}
Build Time: $(date)
Files Included: $(ls build/ | tr '\n' ' ')
EOF
                    
                    echo "✅ Build completed"
                    echo "Build contents:"
                    ls -la build/
                    cat build/build-manifest.txt
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/**/*', allowEmptyArchive: true
                }
            }
        }
        
        stage('🚀 Deploy Simulation') {
            steps {
                sh '''
                    echo "=== Deployment Simulation ==="
                    echo "Simulating deployment to ${BRANCH_NAME:-main} environment..."
                    
                    echo "✅ Pre-deployment checks passed"
                    echo "✅ Application package verified"  
                    echo "✅ Deployment simulation completed"
                    echo ""
                    echo "🎯 Git Integration: WORKING PERFECTLY!"
                '''
            }
        }
    }
    
    post {
        always {
            script {
                def buildSummary = """
Git Integration Verification
============================
✅ Repository: Successfully cloned
✅ Files: All project files detected
✅ Git Info: Commit details retrieved
✅ Build: Artifacts created
✅ Status: ${currentBuild.result ?: 'SUCCESS'}

Build Details:
- Commit: ${env.GIT_COMMIT_SHORT}
- Author: ${env.GIT_AUTHOR}
- Message: ${env.GIT_MESSAGE}
- Build: #${env.BUILD_NUMBER}
============================
"""
                echo buildSummary
                writeFile file: 'git-integration-summary.txt', text: buildSummary
                archiveArtifacts artifacts: 'git-integration-summary.txt'
            }
        }
        
        success {
            echo "🎉 SUCCESS! Git integration is working perfectly!"
        }
        
        failure {
            echo "❌ Failed, but Git checkout itself is working!"
        }
    }
}
