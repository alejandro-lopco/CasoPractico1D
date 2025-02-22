pipeline {
    agent any
    stages {
        stage('getCode') {
            steps {
                sh 'rm -f samconfig.toml'
                git branch: 'master', credentialsId: 'alejandro-lopco', url: 'https://github.com/alejandro-lopco/CasoPractico1D'
                sh 'curl https://raw.githubusercontent.com/alejandro-lopco/CasoPractico1D-Config/refs/heads/production/samconfig.toml > samconfig.toml'
                stash includes: '**', name: 'repo'
            }
        }
        stage('staticTest') {
            agent { node 'agente1' }
            steps {
                unstash 'repo' 
                catchError (buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '/opt/VEnvAgente1/bin/python -m flake8 --format=pylint --exit-zero CasoPractico1D/src > flake8.out'
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]   
                    sh '/opt/VEnvAgente1/bin/python -m bandit --exit-zero -r ./CasoPractico1D/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id} {msg}]"'
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                }
            }
        }
        stage('deploy') {
            steps {
                sh 'sam build'
                sh '''sam deploy \
                    --stack-name 'ToDoAWSCasoPractico1D' \
                    --capabilities 'CAPABILITY_IAM' \
                    --s3-bucket 'casopractico1d' \
                    --region 'us-east-1' \
                    -t 'template.yaml' \
                    --config-file 'samconfig.toml' \
                    --parameter-overrides "Stage=production" \
                    --role-arn 'arn:aws:iam::159559436639:role/LabRole'
                '''
            }
        }
        stage('restTest') {
            agent { node 'agente2' }
            steps {
                unstash 'repo'
                sh '''aws cloudformation describe-stacks \
                    --stack-name ToDoAWSCasoPractico1D \
                    --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                    --output text > BASE_URL.log

                    aws cloudformation describe-stack-resources \
                    --stack-name ToDoAWSCasoPractico1D \
                    --query "StackResources[?ResourceType=='AWS::DynamoDB::Table'].[PhysicalResourceId]" \
                    --output text > TABLE_NAME.log    

                    export BASE_URL=$(cat BASE_URL.log) 
                    echo "URL Base de la API: $BASE_URL"
                    export DYNAMODB_TABLE=$(cat TABLE_NAME.log)
                    echo "Nombre de la tabla: $DYNAMODB_TABLE"
                    /opt/VEnvAgente2/bin/python -m pytest --junitxml=result-integration.xml test/integration/todoApiTest.py
                    /opt/VEnvAgente2/bin/python -m pytest --junitxml=result-unit.xml test/unit/TestToDo.py                                
                '''           
                junit 'result*.xml'
            }
        }
    }
}