pipeline {
    agent any
    stages {
        stage('Compile') {
            steps {
                sh 'javac src/Main.java'
                echo 'Compilation successful!'
            }
        }
        stage('Test') {
            steps {
                echo 'Tests will run here after adding JUnit dependency.'
            }
        }
    }
}
