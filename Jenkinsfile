pipeline {
    agent any
        stages {
            stage('Build') {
                steps {
                    sh 'echo This is build'
                    sh 'env'
                }
            }
            stage('Test') {
                steps {
                    sh 'echo This is Test'
                }
            }
            stage('Deploy') {
                steps {
                    sh 'echo This is Deploy'
                }
            }
        }
}    

