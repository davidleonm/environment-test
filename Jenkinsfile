pipeline {
  agent { label 'slave' }

  environment {
      GITHUB_TOKEN = credentials('9ed2fdd8-9883-4ec5-b073-042b2b9b3457')
  }

  stages {
    stage('Prepare Python ENV') {
      steps {
        script {
          SetBuildStatus('pending')

          // Clean & Prepare new python environment
          sh 'rm -rf ENV'
          sh 'python3 -m venv ENV'

          sh 'ENV/bin/pip install --upgrade pip'
          sh "ENV/bin/pip install -r ${WORKSPACE}/PythonHelloWorld/requirements.txt"
        }
      }
    }

    stage('Execute unit tests') {
      steps {
        script {
          sh "ENV/bin/python -m unittest discover -s ${WORKSPACE}/PythonHelloWorld"
        }
      }
    }

    stage('SonarQube analysis') {
      environment {
        def scannerHome = tool 'Sonarqube'
      }
      
      steps {
        script {
          sh "coverage run -m unittest discover -s ${WORKSPACE}/PythonHelloWorld"
          sh "coverage xml -i"
        }
        withSonarQubeEnv('Sonarqube') {
          sh "${scannerHome}/bin/sonar-scanner"
        }
      }
    }
  }
  post {
    success {
      script {
        SetBuildStatus('success')
      }
    }

    failure {
      script {
        SetBuildStatus('failure')
      }
    }
  }
}

def SetBuildStatus(String status) {
  sh "curl -H 'Authorization: Bearer ${GITHUB_TOKEN}' \
      -H 'Content-Type: application/json' \
      -X POST 'https://api.github.com/repos/davidleonm/jenkins-test/statuses/${GIT_COMMIT}' \
      -d '{\"state\": \"${status}\",\"context\": \"continuous-integration/jenkins\", \"description\": \"Jenkins\", \"target_url\": \"${BUILD_URL}\"}'"
}