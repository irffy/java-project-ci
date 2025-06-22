pipeline {
    agent any

    environment {
        // Optional: Set JAVA_HOME explicitly
        JAVA_HOME = '/usr/lib/jvm/java-21-temurin'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Debug: Environment Check') {
            steps {
                echo '🔍 Testing Jenkins using git commit'
                echo '🔍 Checking Java and workspace...'
                sh 'echo JAVA VERSION: && java -version'
                sh 'echo JAVAC VERSION: && javac -version'
                sh 'echo CURRENT DIRECTORY: && pwd'
                sh 'echo LISTING FILES: && ls -R'
            }
        }

        stage('Compile Java Source') {
            steps {
                script {
                    def sourceFile = 'src/Main.java'
                    if (fileExists(sourceFile)) {
                        echo "✅ Found ${sourceFile}, compiling..."
                        sh "javac ${sourceFile}"
                        echo '✅ Compilation successful!'
                    } else {
                        error "❌ File ${sourceFile} not found! Check your repo structure."
                    }
                }
            }
        }

        stage('Run Java Program') {
            steps {
                script {
                    if (fileExists('src/Main.class')) {
                        echo '▶️ Running Main.class...'
                        sh 'java -cp src Main'
                    } else {
                        error '❌ Main.class not found. Compilation may have failed.'
                    }
                }
            }
        }

        stage('Test Placeholder') {
            steps {
                echo '🧪 Placeholder: Add JUnit tests here later.'
            }
        }
    }

    post {
        success {
            echo '✅ Build completed successfully!'
        }
        failure {
            echo '❌ Build failed. Check console output for errors.'
        }
        always {
            echo '📦 Archiving workspace contents...'
            sh 'mkdir -p archive && cp -r src/*.class archive/ || true'
            archiveArtifacts artifacts: 'archive/*.class', allowEmptyArchive: true
        }
    }
}
