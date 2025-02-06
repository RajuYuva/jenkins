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
        text(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        string(name: 'BIOGRAPHY', defaultValue: 'Hellow how are you', description: 'Enter some information about the person')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
        file(name: 'CONFIG_FILE', description: 'Upload configuration file')

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
        stage("print params"){
            steps{
                echo "Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                echo "Password: ${params.PASSWORD}"
                echo "triggered test again"
                //error 'some failure'
            }
        }
    }
}

