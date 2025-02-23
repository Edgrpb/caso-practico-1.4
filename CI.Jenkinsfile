pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('GITHUB_TOKEN')
    }
    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/Edgrpb/caso-practico-1.4.git'
            }
        }
        
        stage('Static Test') {
            steps {
                sh '''
                    cd src
                    python -m flake8 --format=pylint --exit-zero app > flake8.out
                    python -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                '''
                // Publicar informes sin Quality Gates
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env staging --config-file samconfig.toml --resolve-s3 --no-confirm-changeset --capabilities CAPABILITY_IAM
                '''
            }
        }
        
        stage('Rest Test') {
            steps {
                script {
                    def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text", returnStdout: true).trim()
                    echo "Base URL: $BASE_URL"
                    env.BASE_URL = BASE_URL
                    
                    // Ejecutar pruebas con Pytest
                    sh """
                        export BASE_URL=$BASE_URL
                        echo "Ejecutando pruebas con Pytest..."
                        python -m pytest --maxfail=1 ${WORKSPACE}/test/integration/todoApiTest.py
                    """

                    // Ejecutar pruebas con curl t
                    sh """
                        echo "Ejecutando pruebas con curl..."
                        
                        RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $BASE_URL/tareas)
                        if [ "$RESPONSE" -ne 200 ]; then
                            echo "Error: La API devolvió código $RESPONSE"
                            exit 1
                        fi
                    """
                }
            }
        }
        
        stage('Promote') {
            steps {
                script {
                    sh '''
                        git fetch --all
                        git checkout master
                        git merge origin/develop
                        git push https://${GITHUB_TOKEN}@github.com:Edgrpb/caso-practico-1.4.git
                        
                    '''
                }
            }
        }
    }
}
