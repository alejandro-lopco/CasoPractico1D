pipeline {
    agent any
    stages {
        stage('getCode') {
            steps {
                git branch: 'develop', url: 'https://github.com/alejandro-lopco/CasoPractico1D'
                stash includes: '**', name: 'repo'
                sh 'ls'
            }
        }
        stage('staticTest') {
            steps {
                catchError (buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '/opt/CasoPracticoVEnv/bin/python -m flake8 --format=pylint --exit-zero src > flake8.out'
                    sh 'cat flake8.out'
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]   
                    sh '/opt/CasoPracticoVEnv/bin/python -m bandit --exit-zero -r ./CasoPractico1D/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id} {msg}]"'
                    sh 'cat bandit.out'
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                }
            }
        }
        stage('deploy') {
            steps {
                catchError (buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'sam build'
                    sh '''sam deploy \
                        --stack-name 'ToDoAWSCasoPractico1D' \
                        --capabilities 'CAPABILITY_IAM' \
                        --s3-bucket 'casopractico1d' \
                        --region 'us-east-1' \
                        -t 'template.yaml' \
                        --config-file 'samconfig.toml' \
                        --parameter-overrides "Stage=staging" \
                        --role-arn 'arn:aws:iam::159559436639:role/LabRole'
                    '''
                }
            }
        }
    }
}