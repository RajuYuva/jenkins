// Advanced parameters (Git Parameter and Active Choices must be inside properties([]))
properties([
    parameters([
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            branchFilter: 'origin/*',
            defaultValue: 'master',
            description: 'Select a branch to build'
        ),
        // activeChoice(
        //     name: 'ENVIRONMENTS',
        //     choiceType: 'CHECKBOX', // Use 'SINGLE_SELECT' for dropdown, 'CHECKBOX' for multi-select
        //     description: 'Select environments for deployment',
        //     script: [
        //         $class: 'org.biouno.unochoice.model.GroovyScript',
        //         fallbackScript: 'return ["default"]', // Used if script execution fails
        //         script: 'return ["dev", "staging", "prod"]' // Script returning values
        //     ]
        // )
    ])
])

pipeline {
    
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 1, unit: 'SECONDS')
    }
    agent any
    
    parameters {
        //Supported parameter types
        booleanParam(name: 'buildnagaSolrDocker', defaultValue: false, description: 'Do you want to build naga-solr-docker image?')
        booleanParam(name: 'buildSolrDockerClassification', defaultValue: false, description: 'Do you want to build solr-docker-classification image?')
        booleanParam(name: 'buildSolrDockerVisualmatch', defaultValue: false, description: 'Do you want to build solr-docker-visualmatch image?')

        text(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        string(name: 'BIOGRAPHY', defaultValue: 'Hello how are you', description: 'Enter some information about the person')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')

        file(name: 'CONFIG_FILE', description: 'Upload configuration file')

        run(name: 'UPSTREAM_BUILD', description: 'Select a build from the parent job')

        credentials(name: 'GIT_CREDENTIALS', description: 'Select Git credentials')
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

