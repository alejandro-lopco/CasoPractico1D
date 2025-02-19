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
                    sh 'python -m flake8 --format=pylint --exit-zero src > flake8.out'
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]   
                    sh 'python -m bandit --exit-zero -r ./CasoPractico1D/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id} {msg}]"'
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                    sh 'python -m coverage run --branch --source=src --omit=casoPractico1D/src/__init__.py,casoPractico1D/app/requirements.txt,casoPractico1D/app/decimalencoder.py -m pytest casoPractico1D/test/unit/TestToDo.py'
                    sh 'python -m coverage xml'
                    cobertura coberturaReportFile: 'coverage.xml'
                }
            }
        }
    }
}