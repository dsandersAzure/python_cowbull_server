def major = '2'
def minor = '2'

podTemplate(containers: [
    containerTemplate(name: 'redis', image: 'k8s-master:32080/redis:5.0.3-alpine', ttyEnabled: true, command: 'redis-server'),
    containerTemplate(name: 'python', image: 'k8s-master:32080/python:3.7.4-alpine3.10', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'jre', image: 'k8s-master:32080/openjdk:11-jre', ttyEnabled: true, command: 'cat', envVars: [envVar(key: 'dns', value: '8.8.8.8')]),
  ]) {
  node(POD_LABEL) {
    stage('Verify Redis is running') {
        container('redis') {
            sh 'redis-cli ping'
        }
    }
    stage('Setup environment') {
        git 'https://github.com/dsandersAzure/python_cowbull_server'
        container('python') {
            sh """
                python --version
                python3 -m pip install -q -r requirements.txt
            """
        }
    }
    stage('Execute Python unit tests') {
        container('python') {
            sh """
                export PYTHONPATH="\$(pwd)/:\$(pwd)/unittests"
                coverage run -m unittest unittests
                coverage xml -i
            """
        }
    }
    stage('Execute Python system tests') {
        container('python') {
            sh """
                export PYTHONPATH="\$(pwd)/:\$(pwd)/systests"
                export PERSISTER='{"engine_name": "redis", "parameters": {"host": "localhost", "port": 6379, "db": 0, "password": ""}}'
                export LOGGING_LEVEL=30
                python -m unittest systests
            """
        }
    }
    stage('Sonarqube code coverage') {
        container('jre') {
            def scannerHome = tool 'SonarQube Scanner';
            withSonarQubeEnv('Sonarqube') { // If you have configured more than one global server connection, you can specify its name
                sh """
                    rm -rf *.pyc
                    rm -f /var/jenkins_home/workspace/cowbull-server/.scannerwork/report-task.txt
                    rm -f /var/jenkins_home/workspace/cowbull-server/.sonar/report-task.txt
                    echo "Run sonar scanner"
                    chmod +x ${scannerHome}/bin/sonar-scanner
                    ${scannerHome}/bin/sonar-scanner -X -Dproject.settings=./sonar-project.properties -Dsonar.python.coverage.reportPath=./coverage.xml -Dsonar.projectVersion="${major}"."${minor}"."${env.BUILD_NUMBER}"
                """
            }
        }
    }
    stage('Sonarqube quality gate') {
        sh """
          echo "Still a work in progress :) "
        """
    }
  }
}