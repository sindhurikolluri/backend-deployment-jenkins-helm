pipeline {
    agent { label 'AGENT-1' }
    environment { 
        PROJECT = 'expense'
        COMPONENT = 'backend' 
        DEPLOY_TO = "production"
        REGION = "us-east-1"
        appVersion = ''
        environment = ''
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }
    parameters{
        string(name: 'version',  description: 'Enter the application version')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick something')
    }
    stages {

        stage('Setup Environment'){
            steps{
                script{
                    appVersion = params.version
                    environment = params.deploy_to
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    withAWS(region: 'us-east-1', credentials: "aws-creds-${environment}") {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name expense-${environment}
                            kubectl get nodes
                            cd helm
                            sed -i 's/IMAGE_VERSION/${params.version}/g' values-${environment}.yaml
                            helm upgrade --install $COMPONENT -n $PROJECT -f values-${environment}.yaml .
                        """
                    }
                }
            }
        }
        stage('Functional/API Tests') {
            when{
                expression { params.deploy_to == 'dev'}
            }
            
            steps {
                script{
                    
                        sh """
                            echo "functional tests will be performed after DEV deployment. Usual;y these are automated selenium test cases written by testing team. If these test cases are failed pipeline also fails"
                        """
                    
                }
            }
        }

        stage('Integration Tests') {
            when{
                expression { params.deploy_to == 'qa'}
            }
            
            steps {
                script{
                    
                        sh """
                            echo "integrations tests will be performed after QA/UAT deployment. Usually these are automated BDD(Behaviour driven development) test cases in cucumber framework written by testing team. If these test cases are failed pipeline also fails"
                        """
                    
                }
            }
        }
        
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'I will run when pipeline is failed'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
    }
}