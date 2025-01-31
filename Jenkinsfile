pipeline {
    
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 1, unit: 'SECONDS')
    }
    agent {
        label 'AGENT-1'
    }
    parameters {
        booleanParam(name: 'buildAlamySolrDocker', defaultValue: false, description: 'Do you want to build alamy-solr-docker image?')
        booleanParam(name: 'buildSolrDockerClassification', defaultValue: false, description: 'Do you want to build solr-docker-classification image?')
        booleanParam(name: 'buildSolrDockerVisualmatch', defaultValue: false, description: 'Do you want to build solr-docker-visualmatch image?')
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
                sh 'sleep 10'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo This is Deploy'
            }
        }
    }
}

