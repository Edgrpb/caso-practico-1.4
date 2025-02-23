pipeline {
    agent any
    environment {
        TOKEN_ID = credentials('TOKEN_ID')
    }
    
    stages {
        //descargar codigo
        stage('Get Code') {
            steps {
                git branch: 'master', 
                    credentialsId: 'TOKEN_ID',
                    url: 'https://github.com/usuario/repositorio.git'
            }
        }
        //realizar deploy en Produccion
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env production --config-file samconfig.toml --resolve-s3
                '''
            }
        }
        
        stage('Rest Test') {
            steps {
                script {
                    def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text", 
                        returnStdout: true).trim()
                    
                    echo "Base URL obtenida: $BASE_URL"
                    
                    // Asignar BASE_URL como variable global
                    env.BASE_URL = BASE_URL
                    
                    sh """
                        echo "BASE_URL en pytest: $BASE_URL"
                        BASE_URL=$BASE_URL python -m pytest ${WORKSPACE}/test/integration/todoApiTest.py
                    """
                }
            }
        }
    }
}
