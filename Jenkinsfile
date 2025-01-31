pipeline {
    agent any
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 1, unit: 'SECONDS')
    }
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

